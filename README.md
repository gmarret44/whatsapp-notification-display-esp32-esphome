# WhatsApp Notification Display esp32 esphome

A small ESP32-based display that shows the latest WhatsApp group message on a physical screen, with a sound alert on a passive buzzer — powered by Home Assistant and [ESPHome](https://esphome.io/).

When a message is posted in a chosen WhatsApp group, an automation grabs the message text, pushes it to the ESP32 screen, and optionally plays a short melody on the buzzer.

```
WhatsApp group  →  ha-whatsapp integration  →  Home Assistant automation  →  ESP32 display + buzzer
```

## Hardware

- Generic ESP32 board
- ST7735 SPI TFT display (128x160)
- Passive buzzer
- Push button  (just to run a few actions if you want)

### Wiring

| Component      | Pin    |
|-----------------|--------|
| Display CLK     | GPIO18 |
| Display MOSI    | GPIO23 |
| Display CS      | GPIO22 |
| Display DC      | GPIO21 |
| Display RESET   | GPIO19 |
| Push button     | GPIO15 |
| Buzzer          | GPIO16 |

> ⚠️ GPIO15 is an ESP32 strapping pin. The push button works fine, but avoid adding external pull-up/pull-down resistors on it — this can interfere with boot.

## Software stack

| Layer | What it does | Project |
|---|---|---|
| WhatsApp connection | Connects to WhatsApp Web, exposes messages as Home Assistant sensors, handles group moderation | [FaserF/ha-whatsapp](https://github.com/FaserF/ha-whatsapp) (Add-on + HACS integration) |
| Automation | Watches the "messages received" sensor, filters by group, pushes the text to Home Assistant helpers | `home_assistant/automations.yaml` (this repo) |
| Firmware | Renders the text/image on the screen, plays the buzzer melody | `esp32-firmware/display.yaml` (this repo) |

## Repository structure

```
.
├── esp32-firmware/
│   ├── display.yaml           # ESPHome configuration
│   ├── secrets.yaml.example   # Template for Wi-Fi / API secrets
│   └── images/
│       └── dark-vador.jpg     # Example static image
├── home_assistant/
│   ├── helpers.yaml            # input_text / input_select helpers
│   └── automations.yaml        # WhatsApp -> display automation
└── README.md
```

## Setup

### 1. Install the WhatsApp integration

Follow the installation guide at [FaserF/ha-whatsapp](https://github.com/FaserF/ha-whatsapp):
1. Install the **WhatsApp Gateway** add-on (Settings → Add-ons → Add-on Store → Repositories → add the repo URL).
2. Install the **Home Assistant WhatsApp** integration via HACS.
3. Link your WhatsApp account by scanning the QR code.
4. Note the entity ID of your `sensor.whatsapp_<number>_messages_received` sensor, and the ID of the WhatsApp group you want to watch (`123456789@g.us`). The integration includes a built-in tool to find group IDs without digging through logs.

### 2. Add the Home Assistant helpers

Copy the content of [`home_assistant/helpers.yaml`](home_assistant/helpers.yaml) into your `configuration.yaml` (or add them as UI helpers under **Settings → Devices & Services → Helpers**).

### 3. Add the automation

Copy [`home_assistant/automations.yaml`](home_assistant/automations.yaml) into your automations, updating:
- the `entity_id` of the WhatsApp sensor
- the group ID in the `last_sender` condition

### 4. Flash the ESP32

1. Go to `esp32-firmware/`.
2. Copy `secrets.yaml.example` to `secrets.yaml` and fill in your Wi-Fi credentials and a generated API encryption key.
3. Flash with ESPHome:
   ```bash
   esphome run display.yaml
   ```
4. The device shows up automatically in Home Assistant via the ESPHome integration.

## Features

- Displays the latest WhatsApp group message on the screen, with automatic word-wrapping (words are never cut mid-line)
- Displays a static image instead of text when a Home Assistant `input_select` is set to a specific value
- Plays a short melody on the buzzer via a Home Assistant button entity
- Physical push button exposed as a binary sensor in Home Assistant

## Notes

- The `st7735` ESPHome component is deprecated in favor of `mipi_spi`; this project still uses `st7735` for simplicity.
- The WhatsApp integration is a third-party, unofficial client for WhatsApp Web — it is not affiliated with or endorsed by WhatsApp/Meta. Use at your own risk regarding account restrictions.

## License

MIT — see [LICENSE](LICENSE). This does not apply to the third-party `ha-whatsapp` project, which has its own license.
