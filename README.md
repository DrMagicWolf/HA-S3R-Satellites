# KITT satellite firmware (M5Stack Atom Echo S3R)

```
kitt/
├── core.yaml                  voice assistant, Sendspin multi-room, audio chain, button gestures, LED dispatcher
├── hw-echo-s3r.yaml           Atom Echo S3R pins / ES8311 codec
├── led-none.yaml              no status LED (stock S3R)
├── led-grove-ws2812.yaml      one SK6812/WS2812 on the Grove port
├── echo-s3r.factory.yaml      FACTORY IMAGE — stock S3R
└── echo-s3r-led.factory.yaml  FACTORY IMAGE — S3R with Grove LED
```

## One-time: publish the packages
1. Create a **public** GitHub repo (e.g. `kitt-satellites`) and push this `kitt/` folder.
2. In both `*.factory.yaml`, replace `REPO_OWNER/REPO_NAME` with yours. Push.

## Per device
1. ESPHome Builder → New Device → Continue → name `kitt-factory` (any) → ESP32-S3 → Skip.
   Replace its YAML with **one line**:
   `packages: { kitt: github://REPO_OWNER/REPO_NAME/kitt/echo-s3r.factory.yaml@main }`
   (or `echo-s3r-led.factory.yaml`). This entry is only used to build the factory image.
2. Install → Plug into this computer → hold the S3R side button ~2 s until the green
   LED lights → pick the port. Enter Wi-Fi when the installer asks (Improv).
3. Builder home page: the unit appears under **Discovered** as `kitt-echo-s3r-xxxxxx` → **Adopt**.
   Builder writes `<name>.yaml` with API key, OTA password and `!secret wifi_*`.
4. Edit the adopted file: set `name`/`friendly_name` under `substitutions`
   (e.g. `guest-room-south-va`), Install (OTA).
5. HA → Settings → Voice assistants → the satellite → pipeline **KITT (HA)**, wake word **Hey KITT**.
   Music Assistant → the "Music" player joins Sendspin groups like any other.

Repeat 2–5 for every S3R. Firmware changes are made once in the repo; each device
picks them up on its next Install.
