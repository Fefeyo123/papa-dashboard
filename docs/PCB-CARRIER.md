# PCB Carrier

Hardware-notities voor de carrier-PCB van het dashboard.

## Modules

- ESP32 DevKit V1 / 30-pin clone
- Waveshare 7.5 inch e-paper HAT / driver board
- TP4056 laadmodule met batterijbescherming
- MT3608 boost converter
- MAX17043 fuel gauge breakout
- 1S LiPo
- 3-pin slide switch

## ESP32 Pinout

E-paper:

| Signaal | ESP32 |
| --- | --- |
| `DIN` | `GPIO14` |
| `CLK` | `GPIO13` |
| `CS` | `GPIO15` |
| `DC` | `GPIO27` |
| `RST` | `GPIO26` |
| `BUSY` | `GPIO25` |
| `VCC` | `3V3` |
| `GND` | `GND` |

MAX17043:

| Signaal | ESP32 |
| --- | --- |
| `SDA` | `GPIO21` |
| `SCL` | `GPIO22` |
| `VCC` | `3V3` |
| `GND` | `GND` |

## Voedingspad

```text
LiPo -> TP4056 -> slide switch -> MT3608 -> ESP32 VIN
```

De TP4056 moet de versie met bescherming zijn. Op veel modules herken je die
aan aansluitingen zoals `B+`, `B-`, `OUT+` en `OUT-`.

## MAX17043 Plaatsing

Voor een echte uit-stand hoort de `+` meetpin van de MAX17043 op de geschakelde
batterijrail:

```text
MAX17043 + -> dezelfde net als MT3608 VIN+
MAX17043 - -> GND / TP4056 OUT-
```

Dus niet rechtstreeks op `TP4056 OUT+` voor de schakelaar. Anders kan de ESP32
via I2C of de gauge deels teruggevoed worden terwijl de schakelaar uit staat.

## Slide Switch

Gebruik een 3-pin slide switch als eenvoudige high-side schakelaar:

| Switch pin | Verbinding |
| --- | --- |
| midden | `TP4056 OUT+` |
| buitenpin 1 | `MT3608 IN+` en `MAX17043 +` |
| buitenpin 2 | niet aangesloten |

`TP4056 OUT-`, `MT3608 IN-`, `MAX17043 -` en ESP32 `GND` zitten samen op ground.

## Belangrijke Netten

| Net | Verbindingen |
| --- | --- |
| `VBAT_PROTECTED` | `TP4056 OUT+`, switch midden |
| `VBAT_SWITCHED` | switch uitgang, `MT3608 IN+`, `MAX17043 +` |
| `GND` | alle grounds |
| `5V_BOOST` | `MT3608 OUT+`, ESP32 `VIN` |
| `3V3` | ESP32 `3V3`, e-paper `VCC`, MAX17043 `VCC` |
| `I2C_SDA` | ESP32 `GPIO21`, MAX17043 `SDA` |
| `I2C_SCL` | ESP32 `GPIO22`, MAX17043 `SCL` |

## Meetpunten Die Handig Zijn

- `TP4056 OUT+` tegenover `GND`: beschermde batterijspanning
- `VBAT_SWITCHED` tegenover `GND`: alleen spanning als de schakelaar aan staat
- `MT3608 OUT+` tegenover `GND`: boost-uitgang
- ESP32 `3V3` tegenover `GND`: regulator-uitgang van het dev board

## Bekende Trade-Off

Een ESP32 dev board is makkelijk te flashen, maar niet ideaal voor ultra-low
power. De power LED, USB-serial chip en onboard regulator verbruiken ook in deep
sleep. Voor deze versie is dat acceptabel, maar een volgende PCB kan zuiniger
worden met een kale ESP32-module en een betere 3.3V-regulator.
