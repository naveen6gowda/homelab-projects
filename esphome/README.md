# ESPHome Configurations

All devices integrate with Home Assistant via MQTT and/or native API.

## Usage

1. Copy the relevant `.yaml` file into your ESPHome config directory
2. Create a `secrets.yaml` with your credentials:

```yaml
wifi_ssid: "YourNetwork"
wifi_password: "YourPassword"
ota_password: "generate_a_random_string"
api_key: "generate_with_esphome_cli"
```

3. Replace placeholder values in the YAML with `!secret` references
4. Flash via ESPHome dashboard or `esphome run <file>.yaml`

## Device Map

| File | Board | Location | Sensors |
|------|-------|----------|---------|
| `mailbox-alert.yaml` | ESP32-C6 SuperMini | Mailbox | Reed switch, AHT21 |
| `plant-moisture.yaml` | ESP32-C3 | Plant pot | Capacitive moisture, OLED |
| `hall-clock.yaml` | ESP32-C3 | Hallway | LD2410C radar, OLED, BME280 |
| `touch-voice-assistant.yaml` | ESP32-S3 16MB | Living room wall | Touchscreen, I2S mic+speaker |
| `bathroom-monitor.yaml` | ESP32-C3 | Bathroom | BME280, battery voltage |
| `livingroom-monitor.yaml` | ESP32-C3 | Living room | AHT20, ENS160 CO2/AQI, OLED |
| `kitchen-display.yaml` | ESP32-S3 | Kitchen | Touch display, relay control |
| `office-monitor.yaml` | ESP32-C3 SuperMini | Office | Multi-sensor |
| `epaper.yaml` | ESP32-C3 | Any | E-ink display |

## Notes

- All API keys and OTA passwords in these files are **placeholders** — generate your own
- WiFi credentials replaced with `YOUR_WIFI_SSID` / `YOUR_WIFI_PASSWORD`
- Deep sleep devices (mailbox, plant moisture) require special OTA procedure — see comments in YAML
