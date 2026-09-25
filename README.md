![picokit-50-finale](https://raw.githubusercontent.com/mytechnotalent/picokit-50-finale/main/picokit-50-finale.png)

<br>

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/reverse-engineering)
## FREE Embedded Hacking Course [HERE](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# PICOKIT-50 FINALE

### Full Peripheral Capstone and Authenticated Telemetry
#### Lesson 50 of the Picokit Series

<br>

***
**LEGAL DISCLAIMER:**
The information, tools, and code provided in this repository and course are strictly for educational, research, and defensive purposes only.

You are explicitly prohibited from using any materials contained herein to access, test, modify, or exploit any device, network, or system that you do not own 100% or for which you do not have explicit, documented, and legally binding authorization to interact with.

By using this repository and course, you acknowledge and agree that:

1. Any illegal, unauthorized, or malicious use of this information is solely your responsibility.
2. The author(s) and contributor(s) of this repository and course shall not be held liable for any damages, legal repercussions, criminal charges, or unauthorized actions resulting from the use, misuse, or abuse of the contents herein.
3. You will comply with all applicable local, state, national, and international laws regarding cybersecurity and computer fraud.

**IF YOU DO NOT AGREE WITH THESE TERMS, DO NOT USE THIS REPOSITORY AND COURSE.**
***

<br>
<br>

## Overview

The capstone of the Picokit series. One node combines the DHT11, the red,
yellow, and green annunciator LEDs, the 1602 LCD, the SG90 servo, the
push-button, the VS1838B infrared remote, and two-way LoRa into a single
authenticated telemetry node. Every reading is shown on the LCD, latched and
annunciated on the LEDs, and sealed into the heartbeat that the gateway
authenticates before it trusts a single byte.

<br>

## What it teaches

- Combining every peripheral from the series into one paced state machine.
- A heartbeat that carries both readings: `{"n":50,"s":<seq>,"t":<t>,"h":<h>}`.
- Two-way LoRa: authenticated heartbeats out, `+RCV` commands in.
- The gateway side: receive, authenticate, reject, log, and display.

<br>

## Hardware

| Peripheral | Pico 2 pin | Role |
| --- | --- | --- |
| DHT11 | GP4 | temperature and humidity |
| Red / Yellow / Green | GP16 / GP17 / GP18 | annunciator |
| 1602 LCD | GP2 SDA / GP3 SCL | local readout |
| SG90 servo | GP14 | damper or latch |
| Push-button | GP15 | local acknowledge |
| VS1838B IR | GP5 | remote acknowledge |
| Onboard LED | GP25 | heartbeat |
| RYLR998 | GP8 TX / GP9 RX | two-way LoRa |
| Debug Probe | SWCLK/SWDIO/GND, GP0/GP1 | SWD and the console |

<br>

## How it works

The node runs `monitor_step` in a loop. Every 2 seconds it samples the DHT11,
latches the safe band, drives the servo, prints the state, and renders the
reading to the LCD, and every 5 seconds it seals
`{"n":50,"s":<seq>,"t":<tenths>,"h":<tenths>}` with the field key and sends it
over LoRa. The button and the infrared remote both acknowledge the latch, and
inbound `+RCV` lines are pumped every tick. The gateway authenticates each
frame and only then parses it.

<br>

## Build and flash

```bash
cd firmware
cmake -S . -B build -G Ninja -DPICO_BOARD=pico2 -DPICO_PLATFORM=rp2350-arm-s
cmake --build build
openocd -f interface/cmsis-dap.cfg -f target/rp2350.cfg \
  -c "program build/picokit_50_finale.elf verify reset exit"
```

<br>

## Watch the node

Open the console at 115200 and reset:

```text
BOOT
I2C scan:
  no devices
=== PICOKIT-50 FINALE // FULL PERIPHERAL + AUTHENTICATED HEARTBEAT ===
TEMP 230 ALARM 0 DOOR 0
RX from 0x0002, N bytes
ACK
```

<br>

## The gateway

```bash
cd gateway
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python3 listen.py --port /dev/cu.usbserial-A50285BI --hub 0001 --network 18 --db gateway.db
```

It prints `OK node=50 rssi=...` per authenticated heartbeat. The terminal
dashboard `python3 tui.py --db gateway.db` and the web dashboard
`python3 web/app.py --db gateway.db` show the same rows.

<br>

## Verify

```bash
python3 .opencode/skill/embedded-c-standard/audit_c_standard.py
python3 .opencode/skill/embedded-python-standard/audit_python_standard.py
python3 .opencode/skill/iot-readme-standard/validate_readme.py
python3 .opencode/skill/iot-banner-standard/validate_banner.py
python3 scripts/run_tests.py
python3 scripts/check_coverage.py
```

<br>

# Next
[Embedded Hacking](https://github.com/mytechnotalent/Embedded-Hacking)

<br>

# License
[MIT License](https://github.com/mytechnotalent/picokit-50-finale/blob/main/LICENSE)
