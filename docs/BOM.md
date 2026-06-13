# BOM

Onderdelen voor de huidige batterijversie van het Mannenbastion Dashboard.

## Hoofdonderdelen

| Onderdeel | Opmerking |
| --- | --- |
| ESP32 DevKit V1 / 30-pin clone | huidige firmware gebruikt een gewoon `esp32dev` board |
| Waveshare 7.5 inch e-Paper HAT / driver board | `800x480`, V2/V2p-paneel |
| 1S LiPo, bijvoorbeeld 3000 mAh | voeding voor het dashboard |
| TP4056 laadmodule met bescherming | liefst versie met `B+`, `B-`, `OUT+`, `OUT-` |
| MT3608 boost converter | boost naar de ESP32 `VIN` rail |
| MAX17043 fuel gauge breakout | batterijpercentage via I2C |
| 3-pin slide switch | schakelt de batterijrail naar de boost converter |
| Custom carrier PCB | verbindt ESP32, display, voeding en fuel gauge |

## Kleine Onderdelen

- JST-connector of degelijke batterijconnector
- pinheaders of sockets voor modules
- korte, stevige draadjes voor voeding
- montagegaten, afstandsbussen of een frame

## Pinmapping Firmware

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

## Let Op

- `GPIO15` is een ESP32 strapping pin. De huidige hardware werkt ermee, maar
  voeg daar geen extra pull-up of pull-down aan toe zonder opnieuw te testen.
- De `MAX17043 +` hoort op de geschakelde batterijrail als je een echte
  uit-stand wilt.
- Een ESP32 dev board verbruikt in deep sleep meer dan een kale ESP32-module,
  vooral door regulator, USB-chip en power LED.
