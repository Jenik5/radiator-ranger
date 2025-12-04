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
