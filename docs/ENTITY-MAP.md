# Entity Map

De ESP32 leest Home Assistant via een enkele template sensor:

```text
sensor.system_state_cache
```

Die sensor staat in
[`home_assistant/system_state_cache.yaml`](../home_assistant/system_state_cache.yaml).

## Attributes Die De ESP32 Gebruikt

| Attribute | Bron in Home Assistant | Getoond als |
| --- | --- | --- |
| `weather_text` | `weather.gentbrugge` | weertekst |
| `weather_temp` | `weather.gentbrugge` attribute `temperature` | buitentemperatuur |
| `living_temp` | `sensor.anna_temperature` | binnentemperatuur |
| `living_humidity` | `sensor.downstairs_humidity` | luchtvochtigheid |
| `power_now` | `sensor.p1_meter_active_power` | verbruik nu |
| `solar_now` | `sensor.solaredge_current_power` | zon nu |
| `home_battery` | `sensor.solaredge_battery_level` (pas aan) | laadniveau thuisbatterij |
| `home_battery_power` | `sensor.solaredge_battery_power` (pas aan) | laadt / ontlaadt / in rust |
| `gate_open` | `input_boolean.poort_open` | poort open/gesloten |
| `lights_on` | lijst van `light.*` entities | aantal lichten aan |

## Iets Wijzigen

Wijzig bijna altijd de rechterkant van de template, niet de attributenaam.

Voorbeeld:

```yaml
living_temp: "{{ states('sensor.andere_temperatuur') | float(0) }}"
```

De ESP32 blijft dan gewoon `living_temp` lezen.

## Controle In Home Assistant

Ga naar `Developer Tools -> States`, zoek `sensor.system_state_cache`, en kijk
of de attributes gevuld zijn. Waarden zoals `unknown`, `unavailable` of lege
velden wijzen meestal op een verkeerde entitynaam.
