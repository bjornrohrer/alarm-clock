# ESP32 Alarm Clock

## What is this?

A DIY smart alarm clock built from scratch using an **ESP32 microcontroller** and **ESP-IDF** (Espressif's official C framework). The clock displays the time on a 7-segment LED display, plays custom MP3 alarm sounds through a speaker, and is controlled from a phone via a web interface served directly by the ESP32 over Wi-Fi.

The project doubles as a hands-on way to learn embedded C programming, FreeRTOS, and hardware interfacing while building something practical — a bedside alarm clock that eliminates the need to bring a phone into the bedroom.

## Tech Stack

- **Language:** C (via ESP-IDF)
- **Framework:** ESP-IDF v5.x with FreeRTOS
- **Hardware:** ESP32 DevKit V1, TM1637 display, DFPlayer Mini, speaker, physical buttons
- **Phone interface:** HTML/CSS/JS served by the ESP32's built-in HTTP server
- **Editor:** Neovim with clangd LSP
- **Build tooling:** ESP-IDF CLI (`idf.py`)

## Project Goals

1. **Learn embedded C** — Write real C code using a professional framework (ESP-IDF), not Arduino abstractions. Understand structs, pointers, memory, and hardware registers in a practical context.
2. **Understand FreeRTOS** — Work with tasks, queues, timers, and semaphores. Learn concurrent programming on real hardware.
3. **Build something useful** — End result is a working alarm clock on the nightstand that replaces the phone alarm.
4. **Phone control without an app** — Set alarms from a phone browser via a web interface hosted on the ESP32. No app store, no cloud, fully local.
5. **Hardware skills** — Wire real components, read datasheets, implement communication protocols (UART, bit-banged serial), and eventually solder a permanent build.

## Repository Structure

```
├── CLAUDE.md              # This file — project overview
├── ROADMAP.md             # Phased build plan with tasks and shopping lists
├── main/
│   ├── CMakeLists.txt
│   └── main.c             # Entry point (app_main)
├── components/
│   ├── tm1637/            # Display driver
│   ├── dfplayer/          # MP3 player driver
│   ├── alarm/             # Alarm logic and state machine
│   └── webserver/         # HTTP server and REST API
├── spiffs/                # Web interface files (HTML/CSS/JS)
├── CMakeLists.txt
└── sdkconfig              # ESP-IDF configuration
```

## Neovim & Tooling Setup

# Source the environment (run once per terminal session)
. ~/esp/esp-idf/export.sh

### Creating a New Project

# Open menuconfig to set Wi-Fi credentials, serial port, etc.
idf.py menuconfig

# Build, flash, and monitor in one go
idf.py -p /dev/ttyUSB0 flash monitor

# Exit the monitor with Ctrl+]

### Build Commands (Cheat Sheet)

```bash
idf.py build                          # Compile the project
idf.py -p /dev/cu.usbserial-110 flash          # Flash firmware to ESP32
idf.py -p /dev/cu.usbserial-110 monitor        # Open serial monitor
idf.py -p /dev/cu.usbserial-110 flash monitor  # Flash + monitor in one step
idf.py menuconfig                     # Open the config menu (TUI)
idf.py fullclean                      # Nuke the build directory and start fresh
idf.py size-components                # See flash/RAM usage per component
idf.py -p /dev/cu.usbserial-110 erase-flash     # Erase flash
```

> **Note:** On Linux the serial port is usually `/dev/ttyUSB0` or `/dev/ttyACM0`. On macOS it's `/dev/cu.usbserial-*` or `/dev/cu.SLAB_USBtoUART`. Run `ls /dev/tty*` with the ESP32 plugged in to find yours.

## Key Design Decisions

- **ESP-IDF over Arduino** — Chosen for learning depth. ESP-IDF exposes FreeRTOS, the event system, and hardware APIs directly.
- **TM1637 display** — Classic alarm clock aesthetic, readable in the dark, only needs 2 GPIO pins.
- **DFPlayer Mini for sound** — Plays MP3s from an SD card with a built-in amplifier. Controlled via UART.
- **Web interface over native app** — No app development needed. The ESP32 serves a mobile-friendly page accessible from any browser on the local network.
- **NTP for timekeeping** — No RTC module needed. The ESP32 syncs time over Wi-Fi and its internal oscillator maintains accuracy between syncs.
