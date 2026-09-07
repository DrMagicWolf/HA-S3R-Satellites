# AtomEchoS3R HA Satellites

ESPHome firmware for M5Stack Atom Echo S3R units used as Home Assistant voice
satellites with Sendspin multi-room music. One factory image, flashed onto any
number of S3Rs, each adopted individually in the ESPHome Device Builder.

Repo: `github.com/DrMagicWolf/HA-S3R-Satellites`
Project id: `atomechos3r.ha-satellite` (`atomechos3r.ha-satellite-led` for the LED build)

```
atomechos3r/
├── core.yaml                  voice assistant, Sendspin, audio chain, button gestures, LED dispatcher
├── hw-echo-s3r.yaml           Atom Echo S3R pins / ES8311 codec
├── led-none.yaml              no status LED (stock S3R)
├── led-grove-ws2812.yaml      one SK6812/WS2812 on the Grove port
├── echo-s3r.factory.yaml      FACTORY IMAGE — stock S3R
└── echo-s3r-led.factory.yaml  FACTORY IMAGE — S3R with Grove LED
```

## Controls (top button, GPIO41)

| Gesture | Action |
|---|---|
| Short press | Wake the assistant (chime, then listen) |
| Double press | Next track |
| Triple press | Previous track |
| Hold ~1 s | Play / pause |
| Hold 3–8 s | Microphone mute toggle (acts on release) |
| Hold 10 s+ | Factory reset |

The side button is the hardware reset/download button and cannot be read by
firmware. Mute is therefore on the top button and on a switch entity in HA.

## Hardware limitation, by design

The Echo S3R has a single I2S bus, so the microphone and the speaker are
mutually exclusive — ESPHome cannot run it full duplex. **While audio is
playing, wake word detection is off**; the button still works, and the wake
word re-arms when playback stops. The firmware tracks this with a `music_active`
flag and releases the microphone before any playback starts; without that, both
drivers deadlock retrying for the bus.

The stock S3R has no user-controllable RGB LED. Use `led-none.yaml`, or wire one
SK6812/WS2812 to the Grove port (PORT.CUSTOM HY2.0-4P: black GND, red 5 V,
yellow G2) and use `led-grove-ws2812.yaml`.

## Adding a device

1. **Factory-builder entry** (once per image variant). ESPHome Builder →
   **+ Create device** → **Empty Configuration** → name it `atomechos3r-factory`.
   Replace its YAML with:

   ```yaml
   packages:
     satellite:
       url: https://github.com/DrMagicWolf/HA-S3R-Satellites
       files: [atomechos3r/echo-s3r.factory.yaml]
       ref: main
       refresh: 0s        # 1d once the firmware is stable
   ```

   Use `echo-s3r-led.factory.yaml` for units with the Grove LED.

2. **Flash.** Install → Plug into this computer. Put the S3R in download mode
   (hold the side button ~2 s until the green LED lights), pick the port. Enter
   Wi-Fi when prompted (Improv). If no Wi-Fi dialog appears, join the device's
   own AP `atomechos3r-ha-satellite-xxxxxx` and use the captive portal at
   `http://192.168.4.1`.

3. **Adopt.** Builder home page → **Discovered** → **Take Control**. Set the name
   (e.g. `guest-room-south-va`) and friendly name. Builder writes the device file
   with its own API key, OTA password and `!secret wifi_*`.

4. **Install** on that new card (On the network), then add the device in HA.

5. **Assign the pipeline.** HA → Settings → Voice assistants → the satellite →
   Assistant **KITT (HA)**. With *Wake word engine location* = **In Home
   Assistant** (the default), the wake word comes from the pipeline's streaming
   wake word (openWakeWord, e.g. **Hey Kitt**) — the device's own *Wake word*
   dropdown stays on "No wake word", which is correct. The on-device
   microWakeWord models (`okay_nabu`) are only used when that select is set to
   *On device*.

   Music Assistant → the **Music** player joins Sendspin groups like any other.

Repeat 2–5 for every S3R. Firmware changes are made once in this repo; each
device picks them up on its next Install.

## Per-device overrides

Set these in the adopted device file's `substitutions:` block.

| Key | Default | Notes |
|---|---|---|
| `name` / `friendly_name` | `atomechos3r-ha-satellite` | set at adoption |
| `volume_max` | `0.85` | M5 caps the S3R at ~0.8 |
| `volume_initial` | `0.5` | |
| `led_pin` | `GPIO2` | LED build only — Grove yellow wire |
| `led_chipset` | `SK6812` | or `WS2812` |
| `led_brightness` | `0.6` | |

## Troubleshooting

- **Boot banner version** — the first log lines print the project version. If it
  isn't what you just pushed, ESPHome built from its cached copy of the repo;
  `refresh: 0s` forces a fetch.
- **`Driver failed to start` / `Parent bus is busy` repeating every second** —
  the microphone and speaker are fighting over the I2S bus. Power-cycle the
  device; if it recurs on the current firmware, capture the log.
- **Assistant answers to silence** — the openWakeWord add-on threshold is too
  low. 0.5 is a sane starting value; 0.0 fires on anything, including the
  silence sent while muted.
