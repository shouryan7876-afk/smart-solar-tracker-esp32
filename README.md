# Smart Solar Tracker: LDR + Servo + ESP32 IoT Dashboard

![ESP32](https://img.shields.io/badge/board-ESP32-blue) ![Arduino](https://img.shields.io/badge/framework-Arduino-00979D) ![License: MIT](https://img.shields.io/badge/license-MIT-green)

A single-axis **solar tracker** built on the ESP32. Two LDR light sensors compare the light on the left and right of a small solar panel, and an SG90 servo turns the panel towards the brighter side. An INA219 measures the panel voltage, current and power, the battery voltage is monitored, and the ESP32 serves a **live web dashboard** on its own Wi-Fi, with no cloud account or internet needed.

MCA mini project, University Institute of Computing, Chandigarh University.

![Block diagram](docs/block_diagram.png)

<!-- After you add your photos to images/, remove this comment line and the closing arrow to show them.
## Photos

| Prototype | Soldered PCB | Dashboard |
|---|---|---|
| ![Prototype](images/prototype.jpg) | ![PCB](images/pcb.jpg) | ![Dashboard](images/dashboard.png) |
-->

## Live demo (no hardware needed)

`docs/demo.html` is the real dashboard page with simulated data. Once GitHub Pages is enabled (see [docs/upload_to_github.md](docs/upload_to_github.md)) it is available at:

`https://<your-username>.github.io/smart-solar-tracker-esp32/demo.html`

## Contents

- [Features](#features)
- [How it works](#how-it-works)
- [Hardware](#hardware)
- [Getting started](#getting-started)
- [Using the dashboard](#using-the-dashboard)
- [Tuning](#tuning)
- [Repository structure](#repository-structure)
- [Safety notes](#safety-notes)
- [Troubleshooting](#troubleshooting)
- [Limitations and future scope](#limitations-and-future-scope)

## Features

- Automatic east-west tracking with dead-band, averaging, step control and angle limits (20°–160°)
- Live power, panel voltage, current, battery voltage and level, both LDR values, servo angle and mode
- Power graph (last 60 s), manual angle slider and **AUTO / MANUAL** switching
- Works without internet: joins your 2.4 GHz Wi-Fi, or creates its own hotspot **SolarTracker** if the Wi-Fi is not found
- One test sketch per module, so every part can be checked before the full build

## How it works

1. Each LDR forms a voltage divider with a 10 kΩ resistor and is read by the ESP32 (average of 16 samples).
2. Every 100 ms the difference `left − right` is calculated. If it is larger than the dead-band (150 out of 4095) the servo moves 2° towards the brighter side, otherwise it stays.
3. The INA219 sits in series with the panel (+) line and gives voltage and current; power = V × I. The battery voltage comes from a 1:1 resistor divider on GPIO33.
4. A web server on the ESP32 serves the dashboard page and a `/data` JSON endpoint that the page reads every second. `/set?angle=` switches to manual mode and `/mode?auto=1` returns to automatic.

## Hardware

Full list: [`hardware/bom.csv`](hardware/bom.csv)

| Component | Qty |
|---|---|
| ESP32 development board | 1 |
| LDR sensors | 2 |
| SG90 servo motor | 1 |
| Small solar panel (5–6 V) | 1 |
| INA219 current sensor | 1 |
| Rechargeable Li-ion battery + TP4056-type charging module | 1 set |
| 10 kΩ resistors (2 for LDRs, 2 for battery divider) | 4 |
| 470–1000 µF capacitor (servo supply) | 1 |
| Small mirror (reflector) | 1 |
| PCB, headers, jumper wires | 1 set |
| Frame, mounting material and base | 1 set |

### Pin summary

| ESP32 pin | Connected to |
|---|---|
| GPIO34 | LDR 1 (left) divider |
| GPIO35 | LDR 2 (right) divider |
| GPIO33 | Battery voltage divider |
| GPIO18 | Servo signal |
| GPIO21 / GPIO22 | INA219 SDA / SCL |
| 3V3 | LDR dividers, INA219 VCC |
| VIN (5V) | Servo power |
| GND | Everything (common ground) |

Detailed wiring and the checklist before powering on: [docs/wiring.md](docs/wiring.md). Pins as CSV: [`hardware/pinout.csv`](hardware/pinout.csv).

## Getting started

1. Install the **Arduino IDE** and the board package **esp32 by Espressif** (board: *ESP32 Dev Module*).
2. Install the libraries **ESP32Servo** and **Adafruit INA219** (with Adafruit BusIO).
3. Build and test the hardware one stage at a time with the sketches in [`firmware/tests/`](firmware/tests): LDR → servo → INA219 → battery. The soldering order is in [docs/build_guide.md](docs/build_guide.md).
4. In `firmware/SolarTracker/`, copy `config.example.h` to `config.h` and enter your 2.4 GHz Wi-Fi name and password. `config.h` is git-ignored, so your password is not uploaded.
5. Open `firmware/SolarTracker/SolarTracker.ino`, upload it, and open the Serial Monitor at 115200 baud.
6. Open the printed address `http://<ip>` on a phone or laptop on the same Wi-Fi.
   If the Wi-Fi is not found, join the hotspot **SolarTracker** (password `12345678`) and open `http://192.168.4.1`.

## Using the dashboard

| Control | What it does |
|---|---|
| Value cards | Power, panel voltage, current, battery, both LDRs, servo angle, mode |
| Graph | Power over the last 60 seconds |
| Angle slider | Switches to MANUAL and moves the panel to that angle |
| Back to AUTO | Returns to light tracking |

Demo tip: set a tilted angle in MANUAL and note the power, then press **Back to AUTO** and see the power rise as the panel turns to the light.

## Tuning

Settings at the top of `firmware/SolarTracker/SolarTracker.ino`:

| Setting | Meaning |
|---|---|
| `INVERT` | Set `true` if the panel turns away from the light |
| `DEADBAND` | Larger value = less jitter, less sensitive |
| `STEP_DEG` | Degrees moved per control cycle |
| `MIN_ANGLE` / `MAX_ANGLE` | Mechanical limits of the panel |
| `BAT_CAL` | Battery calibration = multimeter volts ÷ dashboard volts |

## Repository structure

```
smart-solar-tracker-esp32/
├── README.md
├── LICENSE
├── .gitignore
├── firmware/
│   ├── SolarTracker/          main sketch (SolarTracker.ino, dashboard.h, config.example.h)
│   └── tests/                 01_ldr_test, 02_servo_test, 03_ina219_test, 04_battery_adc_test
├── hardware/
│   ├── bom.csv                bill of materials
│   └── pinout.csv             pin connections
├── docs/
│   ├── block_diagram.png
│   ├── wiring.md
│   ├── build_guide.md         PCB soldering stages, mechanical build, calibration, demo
│   ├── upload_to_github.md
│   └── demo.html              dashboard demo with simulated data (GitHub Pages)
├── images/                    your photos and screenshots
├── report/                    project report (Word)
└── tools/make_demo.py         regenerates docs/demo.html from dashboard.h
```

## Safety notes

- Check the battery polarity with a multimeter before connecting it. Use a protected cell with a charger module that has battery protection.
- Power the servo from the 5 V pin (not 3.3 V) and join all grounds.
- Check that there is no short between 3V3 and GND before plugging in the ESP32.
- The ESP32 is powered through USB in this design. Running it from the battery needs a 5 V boost converter.

## Troubleshooting

| Symptom | Fix |
|---|---|
| Panel turns away from the light | Set `INVERT = true` |
| Servo jitters | Increase `DEADBAND` |
| ESP32 restarts when the servo moves | Check the capacitor and common GND, use a good USB cable |
| `INA219 not found` | Check SDA/SCL and 3.3V/GND, try swapping SDA and SCL |
| Dashboard does not open | Use the same 2.4 GHz network, or join the SolarTracker hotspot |
| Battery reading is off | Adjust `BAT_CAL` using a multimeter |

More details: [docs/build_guide.md](docs/build_guide.md#5-calibrate-and-tune).

## Limitations and future scope

- Single-axis only (east-west). A dual-axis version would also follow the seasonal tilt.
- Data is shown live but not stored. Cloud logging or an SD card can be added.
- A sun-position algorithm can be combined with the LDRs for better tracking on cloudy days.

## Report

The mini project report (Word) is in [`report/`](report).

## Author

**Shouryan**, MCA (AI & ML), University Institute of Computing, Chandigarh University.

Licensed under the [MIT License](LICENSE).
