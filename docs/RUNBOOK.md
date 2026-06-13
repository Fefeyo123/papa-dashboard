# Runbook

Korte checklist voor bouwen, flashen en debuggen.

## Voor Het Flashen

1. Controleer dat `esphome/secrets.yaml` bestaat.
2. Controleer dat Home Assistant `sensor.system_state_cache` heeft.
3. Sluit de ESP32 via USB aan.
4. Zoek de poort, bijvoorbeeld `/dev/cu.usbserial-1130`.
5. Run een config-check:

```bash
esphome config esphome/papa_dashboard.yaml
```

## Flashen

```bash
esphome run esphome/papa_dashboard.yaml --device /dev/cu.usbserial-1130
```

Logs volgen:

```bash
esphome logs esphome/papa_dashboard.yaml --device /dev/cu.usbserial-1130
```

## Verwachte Logs

Bij een normale wake:

```text
Wifi verbonden: data en tijd ophalen voor slaap
Status cache geladen, wacht nog op geldige tijd
Batterijmeter verversen voor finale refresh
Tijd is binnen, finale refresh voor deep sleep
Display refresh start
Display refresh klaar
Slaap ... seconden tot volgend kwartier
```

Bij ontbrekende wifi:

```text
Geen wifi na koude boot: toon setup-QR scherm
```

of:

```text
Geen wifi na wake uit deep sleep: toon setup-QR scherm
```

## Home Assistant Controleren

Ga naar `Developer Tools -> States` en zoek:

```text
sensor.system_state_cache
```

Check of deze attributes bestaan:

```text
weather_text
weather_temp
living_temp
living_humidity
power_now
solar_now
home_battery
home_battery_power
gate_open
lights_on
```

## Wifi Opnieuw Instellen

Als het dashboard geen wifi vindt, toont het na ongeveer een minuut een QR-pagina.

Gebruik:

```text
SSID: Papa Dashboard Setup
Password: verjaardag123
Portal: http://192.168.4.1/
```

## Batterij Controleren

In de logs moet de I2C scan dit tonen:

```text
Found device at address 0x36
```

Als dat ontbreekt:

- controleer `SDA -> GPIO21`
- controleer `SCL -> GPIO22`
- controleer `GND`
- controleer `VCC`
- controleer of de MAX17043 batterijspanning ziet

## Minimale Bring-Up Firmware

Gebruik alleen [`esphome/papa_bringup.yaml`](../esphome/papa_bringup.yaml) als
je de PCB of ESP32 los wilt testen zonder e-ink dashboard.
