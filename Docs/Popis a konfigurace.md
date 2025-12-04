# Radiátor Ranger – Dokumentace ovládacích prvků, senzorů a konfigurace

## Ovládací prvky

<img src="/images/ovladaci_prvky.png" width="300">

### **Boost (minuty)**
Slouží k okamžitému přetlačení regulace termostatu a k aktivaci topení na maximální výkon po zadanou dobu.  
Během Boostu termostat dál normálně reguluje a pokud se mezitím změní režim (např. schedulerem), po ukončení Boostu bude pokračovat v aktuálně platném režimu.

### **Boost nekonečný**
Aktivuje nepřetržité nucené topení na maximální výkon, dokud není režim ručně ukončen.  
Používá se, pokud je třeba topit naplno bez časového omezení.

### **Stav topení (ovládací prvek)**
Otevře detail termostatu, který umožňuje:
- sledovat aktuální teplotu,  
- upravovat cílovou teplotu,  
- zobrazovat stav (topení / nečinný),  
- přepínat režimy.

Speciální interní stavy (např. Boost, větrání) se zde nezobrazují — ty poskytuje textový senzor.

<img src="/images/termostat.png" width="300">

---

## Senzory

<img src="/images/sensory.png" width="300">

### **Požadavek na topení**
Číselná hodnota **0.0 nebo 1.0**:
- **0.0** znamená, že termostat topení nepožaduje,  
- **1.0** znamená aktivní požadavek.

Je to takto navrženo proto, aby bylo možné snadno sečíst požadavky z více radiátorů a zjistit, kolik zařízení chce aktuálně topit.  
Příklad součtového senzoru je uveden v `/HomeAssistant/template.yaml`.

### **Stav topení (textový senzor)**
Podrobný textový popis toho, co termostat právě dělá.  
Zahrnuje například:

- **„Zapnuto – 15,7 °C – Doma“**  
- **„Vypnuto – Boost“**  
- **„Vypnuto – Otevřené okno“**  
- **„Neaktivní – Mimo topnou sezónu“**

Senzor se používá zejména v ovládacích panelech pomocí card-mod (viz `/HomeAssistant/tile.yaml`).

### **Teplota pro regulaci**
Teplota, kterou termostat skutečně používá pro regulaci.  
Zdroje mají tuto prioritu:

1. **Externí teplota** z HA  
2. **Interní prostorová teplota** (fallback)  
3. **Nouzová hodnota 0 °C**

---

## Diagnostika

<img src="/images/diagnostika.png" width="300">

### **Boost aktivní**
Zobrazuje, zda právě probíhá časový nebo nekonečný Boost, případně kolik času zbývá.

### **Otevřené okno (větrání)**
Externí signál z Home Assistantu, již se zpožděním filtrovaný  
(2 min při otevření, 10 s při zavření).

Má přímý vliv na chování termostatu — **převezme řízení a radiátor vypne**.  
Boost má vyšší prioritu, takže jeho aktivace otevřené okno přebije.

### **Teplota prostoru (fallback)**
Interní teplota z DS18B20 v těle zařízení.  
Používá se při výpadku externího senzoru.

### **Teplota radiátoru (diag)**
Interní teplota tělesa radiátoru (DS18B20).  
Zároveň slouží jako ochrana proti zamrznutí.

### **Topná sezóna**
Externí binární signál z Home Assistantu.

Slouží k odlehčení ventilu i aktuátoru v době, kdy je celé topení mimo provoz.  
Mimo topnou sezónu je ventil ponechán **otevřený**, protože zavírání by způsobovalo mechanické namáhání a aktuátor by navíc trvale odebíral přibližně **7 W** energie.

Topná sezóna má vyšší prioritu než Boost — pokud je mimo sezónu, bude ventil otevřený i při aktivním Boostu.

### **Uptime**
Čas od posledního restartu.

### **WiFi RSSI**
Síla WiFi signálu v dBm.

---

## Konfigurace

Většina chování termostatu se nastavuje pomocí bloku `substitutions` v ESPHome YAML konfiguraci.  
Tento blok umožňuje:

- přizpůsobit názvy entit a zařízení,
- napojit termostat na externí senzory a přepínače v Home Assistantu,
- nastavit jas LED indikace,
- definovat teplotní předvolby,
- upravit Boost a ochranné teploty,
- a zpřehlednit konfiguraci pomocí jednotné systematiky názvů.

Použití substitutions výrazně zjednodušuje přenositelnost mezi místnostmi.

---

### Interní a uživatelské názvy zařízení

```yaml
# Interní názvy zařízení
device_internal_name: "radiator-v-predsini"
device_suffix: "_v_predsini"
device_friendly_suffix: " v předsíni"
```

#### Doporučená systematika názvů

- **device_internal_name**  
  Interní název zařízení používaný ESPHome.  
  Doporučený formát:
  - bez mezer a bez diakritiky,
  - **pomlčky místo podtržítek** (ESPHome může mít s podtržítky problémy),
  - obsahuje přímo umístění,
  - příklad: `"radiator-v-predsini"`.

  Tento název se většinou automaticky vytvoří při zakládání jednotky v ESPHome.

- **device_suffix**  
  Sufix používaný v Home Assistantu při generování názvů entit.  
  Podtržítka jsou zde povolena a praktická.  
  Příklad: `"v_predsini"`.

- **device_friendly_suffix**  
  Lidsky čitelné jméno s mezerami a diakritikou.   
  Příklad: `"v předsíni"`.

Díky této systematice se automaticky odvodí i:

```yaml
device_wifi_name: "${device_internal_name}_wifi"
device_friendly_name: "Radiátor ${device_friendly_suffix}"
```

---

### Interval aktualizace diagnostiky

```yaml
device_sampling_time: "10s"
```

- Určuje, jak často se aktualizují stavové informace (WiFi RSSI, uptime…).  
- Hodnota ve formátu `10s`, `30s`, `1min`.  
- Doporučený interval je **10–60 sekund**.

---

### Jas stavových LED

```yaml
led_green_brightness: "0.4"  
led_red_brightness: "0.6"
```

- Rozsah 0.0–1.0 (0 = zhasnuto, 1 = maximální jas).  
- Slouží také k **vyvážení jasu obou LED**, protože různé barvy svítí rozdílně.
- Také je možné snížit jas na minimum v místnostech, kde by to rušilo.

---

### Adresy teplotních čidel DS18B20

```yaml
dallas_diag: "0xab000000836f6328"  
dallas_fallback: "0xd500000083450828"
```

Termostat používá dvě čidla:

- **dallas_diag** – čidlo na radiátoru (diagnostika, ochrana proti zamrznutí)  
- **dallas_fallback** – interní prostorové čidlo (náhradní teplota pro regulaci)

#### Postup při prvním spuštění

Po prvním startu se čidla **nepřihlásí**, protože nejsou známé jejich adresy.  
Adresy je potřeba přečíst z ESPHome logu:

```yaml
[14:43:52.798][C][gpio.one_wire:021]: GPIO 1-wire bus:  
[14:43:52.803][C][gpio.one_wire:022]:   Pin: GPIO4  
[14:43:52.810][C][gpio.one_wire:084]:   Found devices:  
[14:43:52.815][C][gpio.one_wire:086]:     0xd500000083450828 (DS18B20)  
[14:43:52.822][C][gpio.one_wire:086]:     0xab000000836f6328 (DS18B20)
```

Postup:

1. Adresy zapíšeme do substitutions  
2. Provedeme **druhý start**  
3. Ověříme správnost – jedno čidlo stiskneme mezi prsty a sledujeme, kterému senzoru stoupá teplota  
4. Pokud roste teplota druhému senzoru, adresy je potřeba vyměnit

---

### Externí teplotní a stavové entity z Home Assistantu

```yaml
external_temp_sensor: "sensor.teplomer_${device_suffix}_teplota"  
heating_season_entity: "input_boolean.topna_sezona"
open_window_entity: "binary_sensor.senzor_okna_${device_suffix}_otevirani"
#open_window_entity: "input_boolean.dummy_zavreneho_okna"
```

- **external_temp_sensor**  
  Externí prostorová teplota používaná pro regulaci.  
  Pokud dodržujeme systematiku názvů, název se **sám poskládá**.

- **heating_season_entity**  
  Binární entita určující topnou sezónu.  
  Typicky **pomocník (input_boolean)**.

- **open_window_entity**  
  Senzor otevřeného okna.  
  Při dodržení systematiky názvů se dá jednoduše odvodit správný název.  
  Pokud není reálný sensor okna, použije se dummy helper.

---

### Teplotní předvolby

```yaml
preset_temp_away:      "10"  
preset_temp_sleep:     "18"  
preset_temp_home:      "20"  
preset_temp_comfort:   "22"
```

Význam režimů:

- **Away** – útlum pro dlouhodobou nepřítomnost  
- **Sleep** – spánek. Vhodné i pro **krátkodobou nepřítomnost** (např. v práci)  
- **Home** – běžná teplota při pohybu v místnosti  
- **Comfort** – vyšší teplota při odpočinku (sedění, čtení…)

---

### Boost

```yaml
boost_minutes: "120"  
```

- **boost_minutes**  
  Délka časově omezeného Boostu.  
  Lze využít i prakticky, např. pro **sušení věcí** na radiátoru.

---

### Bezpečnostní teplota radiátoru

```yaml
radiator_safe_temp: "5"
```

- **radiator_safe_temp**  
  Minimální teplota radiátoru při větrání.  
  Pod touto hodnotou nehrozí ztráty tepla a zároveň se chrání proti zamrznutí.

---
