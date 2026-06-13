# Mannenbastion Dashboard

Een batterijgevoed e-ink dashboard voor Home Assistant, gebouwd rond een ESP32,
een Waveshare 7.5 inch e-paper scherm en een kleine Home Assistant template
sensor als databron.

Het dashboard toont:

- klok en datum, uitgelijnd op kwartieren
- batterijpercentage van de ESP32
- weer en buitentemperatuur
- binnentemperatuur en luchtvochtigheid
- poortstatus
- huidig verbruik en zonneproductie
- aantal lichten aan
- een feitje van de dag

De ESP32 wordt elk kwartier wakker, haalt data op, ververst het scherm en gaat
daarna opnieuw in deep sleep. Bij een koude boot toont hij eerst een
opstartscherm. Als wifi niet lukt, toont hij een setup-scherm met QR-codes.

## Projectstructuur

```text
esphome/papa_dashboard.yaml        Productiefirmware voor de ESP32
esphome/papa_bringup.yaml          Minimale testfirmware voor PCB/debug
esphome/fonts/                     Fonts voor tekst en iconen
esphome/external_components/       Custom 4-gray Waveshare driver
home_assistant/system_state_cache.yaml
                                   Template sensor voor Home Assistant
docs/BOM.md                        Onderdelenlijst
docs/ENTITY-MAP.md                 Welke Home Assistant entities gebruikt worden
docs/RUNBOOK.md                    Flashen, testen en troubleshooting
docs/PCB-CARRIER.md                Hardware- en PCB-notities
```

## Belangrijkste bestanden

De hoofdfile is [`esphome/papa_dashboard.yaml`](esphome/papa_dashboard.yaml).
Daarin staan de pinnen, wifi fallback, schermlayout, deep sleep en REST-calls.

De Home Assistant cache staat in
[`home_assistant/system_state_cache.yaml`](home_assistant/system_state_cache.yaml).
Die sensor heet `sensor.system_state_cache`. De ESP32 leest bijna alle
Home Assistant-data via die ene sensor uit.

## Eerste Keer Op Windows

Je hebt geen aparte ESP32-IDE nodig. Alles loopt via ESPHome in een terminal.

### 1. Software installeren

Installeer eerst:

- Python 3 voor Windows
- Visual Studio Code
- Git for Windows
- de VS Code extensies `ESPHome` en `YAML`

Bij Python is vooral belangrijk dat `python` en `pip` beschikbaar zijn in
PowerShell. Test dat met:

```powershell
python --version
pip --version
```

Installeer daarna ESPHome:

```powershell
pip install esphome
```

Controleer:

```powershell
esphome version
```

### 2. ESP32 driver

Een ESP32 dev board heeft meestal een USB-serial chip zoals `CH340`, `CP2102`
of `FTDI`. Als Windows geen COM-poort toont wanneer je de ESP32 aansluit,
installeer dan de driver die bij het board hoort.

De COM-poort vind je in Windows via:

```text
Device Manager -> Ports (COM & LPT)
```

Voorbeelden:

```text
COM3
COM5
COM12
```

### 3. Project openen

Open deze projectmap in VS Code. Open daarna een terminal in VS Code:

```text
Terminal -> New Terminal
```

Ga naar de projectmap als je daar nog niet staat:

```powershell
cd pad\naar\papaproject
```

### 4. Eerst de config controleren

```powershell
esphome config esphome\papa_dashboard.yaml
```

Als dit eindigt met `INFO Configuration is valid!`, is de YAML in orde.

### 5. Flashen via USB

Vervang `COM5` door de poort uit Device Manager:

```powershell
esphome run esphome\papa_dashboard.yaml --device COM5
```

Alleen logs volgen:

```powershell
esphome logs esphome\papa_dashboard.yaml --device COM5
```

Als uploaden faalt, druk tijdens het begin van uploaden de `BOOT` knop op de
ESP32 even in. Sommige dev boards hebben dat nodig.

## Home Assistant-data wijzigen

Home Assistant heeft de template sensor al:

```text
sensor.system_state_cache
```

De file [`home_assistant/system_state_cache.yaml`](home_assistant/system_state_cache.yaml)
is de referentie van wat er in Home Assistant staat. Voeg die dus niet opnieuw
toe als de sensor al bestaat; gebruik hem alleen om later te zien welke
attributes het dashboard verwacht.

Als je in Home Assistant iets aan die template wijzigt, herlaad daarna
`Template Entities` via `Developer Tools -> YAML`. De ESP32 hoeft daarvoor niet
opnieuw geflasht te worden.

## Wat Wijzig Je Waar?

| Wat wil je aanpassen? | Waar aanpassen? | Opnieuw flashen? |
| --- | --- | --- |
| Welke lampen geteld worden | Home Assistant template | Nee |
| Welke poort/helper getoond wordt | Home Assistant template | Nee |
| Andere temperatuur- of energiesensor | Home Assistant template | Nee |
| Home Assistant URL | `esphome/papa_dashboard.yaml` | Ja |
| Schermtekst of layout | `esphome/papa_dashboard.yaml` | Ja |
| Pinnen van display of I2C | `esphome/papa_dashboard.yaml` | Ja |
| Token of OTA-wachtwoord | `esphome/secrets.yaml` | Ja |

### Poort wijzigen

De poortstatus komt uit:

```yaml
gate_open: "{{ states('input_boolean.poort_open') }}"
```

Wil je een andere helper of sensor gebruiken, pas dan alleen die regel aan in
`home_assistant/system_state_cache.yaml`.

### Lichten wijzigen

Het aantal lichten komt uit de lijst onder `lights_on:` in
`home_assistant/system_state_cache.yaml`.

Voeg daar Home Assistant `light.*` entities toe of haal ze weg. De ESP32 hoeft
daarvoor niet opnieuw geflasht te worden; bij de volgende refresh leest hij de
nieuwe telling.

### Andere waarden wijzigen

Deze attributes worden door de ESP32 gebruikt:

```text
weather_text
weather_temp
living_temp
living_humidity
power_now
solar_now
gate_open
lights_on
```

Als een naam verandert, moet dezelfde attributenaam in Home Assistant blijven
bestaan. Je wijzigt dus meestal de entity rechts in de template, niet de naam
links.

## ESPHome secrets

Maak lokaal `esphome/secrets.yaml` aan met:

```yaml
ha_authorization: "Bearer <long-lived-access-token>"
ota_password: "<sterk-wachtwoord>"
```

## Flashen

Op Windows gebruik je een `COM`-poort, bijvoorbeeld `COM5`:

```powershell
esphome run esphome\papa_dashboard.yaml --device COM5
```

Op macOS was de poort tijdens ontwikkeling vaak:

```bash
/dev/cu.usbserial-1130
```

macOS flash-commando:

```bash
esphome run esphome/papa_dashboard.yaml --device /dev/cu.usbserial-1130
```

Config check zonder flash:

```bash
esphome config esphome/papa_dashboard.yaml
```

## Batterij en slaapgedrag

De batterij wordt gemeten met een `MAX17043` fuel gauge op I2C:

```text
SDA -> GPIO21
SCL -> GPIO22
```

De ESP32:

- blijft wakker tot wifi, data, tijd en batterij binnen zijn
- toont de tijd afgerond op `00`, `15`, `30` of `45`
- slaapt tot het volgende kwartier
- toont geen opstartscherm na een normale deep-sleep wake
- toont pas na ongeveer een minuut zonder wifi de setup-QR

De fallback-hotspot is:

```text
SSID: Papa Dashboard Setup
Password: verjaardag123
Portal: http://192.168.4.1/
```

## Hardwarepinnen

E-paper:

```text
CLK  -> GPIO13
DIN  -> GPIO14
CS   -> GPIO15
DC   -> GPIO27
RST  -> GPIO26
BUSY -> GPIO25
```

MAX17043:

```text
SDA -> GPIO21
SCL -> GPIO22
```

Zie [`docs/PCB-CARRIER.md`](docs/PCB-CARRIER.md) voor de volledige
hardware-notities.

## Feitje van de dag

Het feitje wordt maximaal een keer per lokale dag opgehaald via:

```text
https://uselessfacts.jsph.pl/api/v2/facts/random?language=en
```

De ESP32 bewaart het feitje in flash, zodat deep sleep het niet wist. Als de
fetch mislukt, probeert hij later opnieuw.

## Zelf de thuisbatterij toevoegen (optioneel)

Wil je de batterij van het huis op het dashboard zien? Dat kan, maar het scherm
heeft maar plaats voor zes tegels. **Kies dus zelf welke tegel je ervoor inruilt**
(weer, binnen, poort, energie, lichten of het feitje van de dag).

Het komt op drie kleine stappen neer:

1. **Data klaarzetten in Home Assistant.** Zet in
   `home_assistant/system_state_cache.yaml` twee extra attributes bij de andere,
   en wijs ze naar de entities van je batterij (laadniveau in `%` en vermogen in
   watt):

   ```yaml
   home_battery: "{{ states('sensor.jouw_batterij_niveau') | float(0) }}"
   home_battery_power: "{{ states('sensor.jouw_batterij_vermogen') | float(0) }}"
   ```

   Herlaad daarna `Template Entities` via `Developer Tools -> YAML`.

2. **De ESP32 die data laten lezen** in `esphome/papa_dashboard.yaml`:
   - maak onder `sensor:` twee template-sensoren `home_battery` en
     `home_battery_power` (kopieer er een van een bestaande zoals `power_now`);
   - lees ze uit in het script `fetch_dashboard_data`, naast de regels voor
     `power_now`:

     ```cpp
     if (attributes["home_battery"]) {
       id(home_battery).publish_state(attributes["home_battery"].as<float>());
     }
     ```

3. **Op het scherm tonen.** Vervang in de `display:`-lambda het teken-blok van de
   tegel die je niet meer wil door een eigen blokje dat `id(home_battery).state`
   toont, en flash daarna opnieuw.

Liever dat iemand dit voor je doet? Geef gewoon door welke tegel weg mag.

## Troubleshooting

Geen COM-poort op Windows:

- probeer een andere USB-kabel, sommige kabels zijn alleen voor laden
- kijk in `Device Manager -> Ports (COM & LPT)`
- installeer de driver voor `CH340`, `CP2102` of `FTDI`, afhankelijk van het board
- trek de ESP32 uit en steek hem opnieuw in

Upload faalt:

- controleer of je de juiste `COM`-poort gebruikt
- sluit eventuele andere serial monitors
- druk kort op `BOOT` wanneer ESPHome begint te uploaden
- probeer een lagere USB-hubloze aansluiting rechtstreeks op de laptop

Geen data op het scherm:

- check of Home Assistant bereikbaar is via `ha_base_url`
- check of het token in `secrets.yaml` klopt
- check of `sensor.system_state_cache` bestaat
- check in Home Assistant of de attributes gevuld zijn

Geen wifi:

- wacht ongeveer een minuut
- scan de setup-QR
- verbind met `Papa Dashboard Setup`
- open `http://192.168.4.1/`
- vul de juiste wifi in

Geen batterijpercentage:

- check of de MAX17043 op `0x36` op de I2C scan verschijnt
- check `SDA`, `SCL`, `GND`, `VCC` en de batterijmeetlijn
- check of de gauge niet achter een verkeerd geschakelde rail hangt

Scherm crasht tijdens refresh:

- de custom 4-gray refresh duurt lang
- daarom staat `CONFIG_ESP_TASK_WDT_TIMEOUT_S` op `12`
- als je aan de driver werkt, test altijd met USB-logs open

Logs stoppen snel:

- dat is normaal na een succesvolle refresh
- de ESP32 gaat daarna in deep sleep
- wacht tot het volgende kwartier of druk op reset om opnieuw logs te zien
