# ESP32 Alarm Clock — Project Roadmap

> **Timeline:** ~20 weeks at ~5 hours/week  
> **Framework:** ESP-IDF (C) with FreeRTOS  
> **Estimated budget:** €20–45 total  
> **Approach:** Buy components incrementally — only get what you need for the current phase.

---

## Phase 1: Foundation & Setup

**Weeks 1–2 · ~10 hours**

**Goal:** Get your development environment running and blink an LED on the ESP32 using ESP-IDF. This proves your toolchain works end-to-end.

### Shopping List

| Item | Price | Where | Note |
|------|-------|-------|------|
| ESP32 DevKit V1 (USB-C preferred) | €5–10 | Amazon.de, Electrokit.com, or Mouser.dk | The brain — dual-core 240MHz, Wi-Fi + Bluetooth built in |
| Breadboard (400 or 830 point) | €3–5 | Amazon.de — search "breadboard kit Arduino" | For prototyping without soldering |
| Jumper wires (M-M and M-F mix pack) | €3–5 | Amazon.de — ELEGOO 120pcs pack is a good choice | You'll use these throughout the entire project |
| USB cable (micro-USB or USB-C to match your board) | €0–3 | You probably own one already | For programming and power |

### Tasks

#### 1. Install ESP-IDF on your computer

Follow Espressif's official "Get Started" guide. Install ESP-IDF v5.x via the installer (Windows) or the install script (Mac/Linux). Set up Neovim with the ESP-IDF

#### 2. Run the "hello_world" example

Open the hello_world example project in ESP-IDF. Build it, flash it to your ESP32, and open the serial monitor to see the output. This validates that your toolchain, USB driver, and board all work.

> 💡 If flashing fails, hold the BOOT button on the ESP32 while uploading. Some boards need this.

#### 3. Run the "blink" example

Flash the blink example to toggle the built-in LED. Read through the code carefully — notice how it uses `gpio_set_direction()` and `gpio_set_level()` instead of Arduino's `pinMode`/`digitalWrite`. This is your first taste of ESP-IDF's C API.

> 💡 Look at how the code uses `vTaskDelay()` from FreeRTOS instead of `delay()`. Understand that the ESP32 runs tasks, not a simple loop.

#### 4. Study the ESP-IDF project structure

Understand the folder layout: `main/` folder, `CMakeLists.txt`, `sdkconfig`, and the component system. Read about how ESP-IDF uses components — this is how you'll organise your alarm clock code later.

> 💡 Think of components like C libraries with their own headers and source files, managed by CMake.

### Concepts You'll Learn

- ESP-IDF project structure & CMake build system
- GPIO control in C (registers vs HAL)
- FreeRTOS basics: tasks, `vTaskDelay`
- Serial monitor for debugging (`printf` / `ESP_LOGI`)

---

## Phase 2: The Display — Show the Time

**Weeks 3–4 · ~10 hours**

**Goal:** Wire up the TM1637 display and show a hardcoded time like 12:34. Then write a driver in C to communicate with the chip.

### Shopping List

| Item | Price | Where | Note |
|------|-------|-------|------|
| TM1637 4-digit 7-segment display module | €2–4 | Amazon.de — packs of 2–4 are cheapest | Classic alarm clock look, has center colon, readable in the dark, adjustable brightness |

### Tasks

#### 1. Wire the TM1637 to the ESP32

Connect VCC→3.3V, GND→GND, CLK→GPIO 22, DIO→GPIO 21. These are arbitrary GPIO choices — any digital pin works. The TM1637 uses a custom 2-wire protocol (not standard I2C, but similar).

> 💡 Double-check your wiring before powering on. Reversed VCC/GND can fry components.

#### 2. Find or write a TM1637 ESP-IDF component

Search the ESP Component Registry or GitHub for "tm1637 esp-idf". There are several open-source drivers. Clone one into your project's `components/` folder. Study how it implements the communication protocol — the clock/data pin toggling, start/stop conditions, and byte transmission.

> 💡 Even if you use someone else's driver, read every line. Understanding bit-banging a serial protocol is a core embedded skill.

#### 3. Display a hardcoded time

Write code in your main `app_main()` function that initialises the display and shows "12:34" with the colon lit. Experiment with brightness levels (the TM1637 supports 8 brightness settings).

> 💡 Create a helper function like `display_time(uint8_t hours, uint8_t minutes)` that converts numbers to segment patterns. You'll reuse this constantly.

#### 4. Add a counting clock

Make the display count up every second using `vTaskDelay(pdMS_TO_TICKS(1000))`. This isn't accurate timekeeping (you'll fix that later with NTP), but it proves the display updates work and gives you practice with FreeRTOS timing.

> 💡 Toggle the colon on/off every second for that authentic alarm clock blink.

### Concepts You'll Learn

- Bit-banging serial protocols in C
- Reading component datasheets (TM1637)
- 7-segment encoding (which segments make which digit)
- ESP-IDF component system — adding external libraries

---

## Phase 3: Wi-Fi & Real Time

**Weeks 5–7 · ~15 hours**

**Goal:** Connect the ESP32 to your home Wi-Fi and sync the clock to real internet time via NTP. Your display should now show the actual current time.

### Shopping List

No new components needed — this phase is all software.

### Tasks

#### 1. Connect to Wi-Fi using ESP-IDF's wifi driver

Use the "station_example" from ESP-IDF examples. This involves initialising the NVS (non-volatile storage), the TCP/IP stack, creating a Wi-Fi station, and handling events. It's more code than Arduino's `WiFi.begin()`, but you'll understand every layer.

> 💡 Store your SSID and password in `sdkconfig` via `menuconfig` rather than hardcoding them. This is the ESP-IDF way.

#### 2. Understand the ESP-IDF event system

ESP-IDF uses an event-driven architecture. Wi-Fi connection, disconnection, and IP assignment are all events you register handlers for. Study `esp_event_handler_register()` and the `WIFI_EVENT` / `IP_EVENT` event bases. Draw the event flow on paper.

> 💡 This event pattern repeats everywhere in ESP-IDF. Understanding it here pays off for everything else.

#### 3. Sync time with SNTP (NTP)

ESP-IDF has a built-in SNTP component. Configure it to connect to `pool.ntp.org`, set your timezone to CET/CEST (Copenhagen), and get the current time into a `struct tm`. Display this real time on the TM1637.

> 💡 Use `setenv("TZ", "CET-1CEST,M3.5.0,M10.5.0/3", 1)` and `tzset()` for Copenhagen time with automatic DST switching.

#### 4. Create a dedicated clock task

Create a FreeRTOS task (`xTaskCreate`) that runs independently and updates the display every 500ms. This is your first real use of concurrent tasks — the clock task runs in parallel with everything else you'll add later.

> 💡 Give this task a clear name and appropriate stack size (2048 bytes is usually enough). Use `ESP_LOGI` for debug logging.

#### 5. Handle Wi-Fi disconnection gracefully

What happens if your router restarts? Add reconnection logic in the Wi-Fi event handler. The clock should keep showing the last known time and reconnect automatically.

> 💡 The time stays accurate even without Wi-Fi for a while — the ESP32's internal oscillator keeps counting. NTP is just for periodic correction.

### Concepts You'll Learn

- ESP-IDF Wi-Fi station mode & event system
- NVS (non-volatile storage) for config
- SNTP time synchronisation
- Timezone handling in C (POSIX TZ strings)
- FreeRTOS task creation and management
- Concurrent programming: multiple tasks running simultaneously

---

## Phase 4: The Sound System

**Weeks 8–10 · ~15 hours**

**Goal:** Wire up the DFPlayer Mini and speaker. Play an MP3 alarm sound triggered by a command from your code.

### Shopping List

| Item | Price | Where | Note |
|------|-------|-------|------|
| DFPlayer Mini MP3 module | €3–6 | Amazon.de — search "DFPlayer Mini" | Plays MP3/WAV from SD card, has built-in 3W amplifier |
| Small speaker (3W, 8 Ohm) | €3–5 | Amazon.de — search "3W 8 Ohm mini speaker Arduino" | Connects directly to DFPlayer speaker pins |
| Micro SD card (8GB is plenty) | €3–5 | Any shop — format as FAT32 | For storing your alarm sound MP3 files |
| 1kΩ resistor | €0.50 | Included in most resistor kits | Required between ESP32 TX and DFPlayer RX for signal conditioning |

### Tasks

#### 1. Prepare the SD card with alarm sounds

Format the SD card as FAT32. Create a folder called "01" and put MP3 files in it named `001.mp3`, `002.mp3`, etc. The DFPlayer is strict about naming conventions — files must be numbered.

> 💡 Download a few free alarm sounds. Include one gentle one and one loud one for testing. Keep them short (10–30 seconds) — your code will loop them.

#### 2. Wire the DFPlayer to the ESP32

Connect DFPlayer VCC→5V (from USB), GND→GND, RX→GPIO 17 (through the 1kΩ resistor!), TX→GPIO 16, SPK_1→speaker+, SPK_2→speaker−. The resistor on the RX line is critical for voltage level compatibility.

> 💡 Keep the DFPlayer power and speaker wires away from the data wires to reduce noise.

#### 3. Implement UART communication with the DFPlayer

The DFPlayer communicates via serial UART at 9600 baud. Configure UART1 on the ESP32 using `uart_driver_install()` and `uart_param_config()`. Then implement the DFPlayer command protocol: each command is a 10-byte packet with start byte, version, length, command, feedback, parameter, and checksum.

> 💡 This is where you write real C. Define a struct for the command packet, write a `send_command()` function that calculates the checksum, and a `play_track()` wrapper. This is great practice for protocol implementation.

#### 4. Test playback with a simple trigger

Write code that plays track `001.mp3` five seconds after boot. Verify volume control works (commands `0x06` to set volume, range 0–30). Test play, pause, and stop commands.

> 💡 Start at volume 15 out of 30. Full volume through a small speaker can be surprisingly loud and distorted.

### Concepts You'll Learn

- UART serial communication in C
- Communication protocols: packet structure, checksums
- Hardware interfacing: voltage levels, signal conditioning
- Struct-based packet construction in C

---

## Phase 5: Alarm Logic & Physical Buttons

**Weeks 11–13 · ~15 hours**

**Goal:** Create the core alarm system: store alarm times, trigger the sound when the time matches, and add physical buttons for snooze and dismiss.

### Shopping List

| Item | Price | Where | Note |
|------|-------|-------|------|
| Tactile push buttons (2–3 pieces, 6mm) | €1–2 | Amazon.de — search "tactile push button 6mm" | For snooze and dismiss — you'll appreciate these at 6am |
| 10kΩ resistors (2–3 pieces, for pull-ups) | €0.50 | Included in most resistor kits | Optional — you can use ESP32's internal pull-ups instead |

### Tasks

#### 1. Design the alarm data structure

Create a C struct for alarms:

```c
typedef struct {
    uint8_t hour;
    uint8_t minute;
    bool enabled;
    bool days[7];
    uint8_t sound_track;
    uint8_t volume;
} alarm_t;
```

Think about how many alarms to support (8 is plenty). Store them in an array.

> 💡 This is a good exercise in C data modelling. Consider how you'll persist these across reboots — you'll use NVS for that.

#### 2. Implement alarm checking logic

In your clock task (or a new dedicated task), compare the current time against all enabled alarms every second. When a match occurs, set a flag and trigger the DFPlayer to start playing. Think about edge cases: what if the ESP32 boots at exactly an alarm time? What about alarms that are only for certain days of the week?

> 💡 Use a simple state machine: `IDLE → RINGING → SNOOZED → IDLE`. This makes the logic clean and debuggable.

#### 3. Wire and debounce the physical buttons

Connect two buttons: one for SNOOZE, one for DISMISS. Use ESP32's internal pull-up resistors (`gpio_set_pull_mode`) so each button just connects the GPIO to GND when pressed. Implement debouncing — buttons bounce electrically and create multiple triggers from one press.

> 💡 Use a GPIO interrupt (`gpio_isr_handler_add`) with a FreeRTOS queue for clean button handling. Don't poll in a loop — that's wasteful. This is a pattern you'll use in many embedded projects.

#### 4. Implement snooze logic

When SNOOZE is pressed during an alarm: stop the sound, wait 5 or 9 minutes (your choice), then trigger again. When DISMISS is pressed: stop completely and reset for the next alarm.

> 💡 Use a FreeRTOS software timer (`xTimerCreate`) for the snooze countdown. This is cleaner than tracking snooze time manually.

#### 5. Save alarms to NVS (persistent storage)

Use ESP-IDF's NVS (Non-Volatile Storage) to save alarm settings so they survive power loss and reboots. Serialise your `alarm_t` array into NVS using `nvs_set_blob()`. Load them on startup with `nvs_get_blob()`.

> 💡 Write helper functions: `save_alarms()` and `load_alarms()`. Call `save_alarms()` whenever an alarm is modified via the web interface.

### Concepts You'll Learn

- State machines in C for managing complex behavior
- GPIO interrupts vs polling
- Button debouncing (hardware vs software)
- FreeRTOS queues for inter-task communication
- FreeRTOS software timers
- NVS (non-volatile storage) for data persistence

---

## Phase 6: Web Interface — The Phone "App"

**Weeks 14–17 · ~20 hours**

**Goal:** Build an HTTP web server on the ESP32 that serves a mobile-friendly page. From your phone's browser, you can add/edit/delete alarms — just like a real alarm clock app.

### Shopping List

No new components needed — this phase is all software.

### Tasks

#### 1. Set up ESP-IDF's HTTP server

Use `httpd_start()` from `esp_http_server.h` to create a web server on port 80. Register URI handlers for `GET /` (serve the webpage), `GET /api/alarms` (return current alarms as JSON), and `POST /api/alarms` (update alarms). Study the "restful_server" ESP-IDF example.

> 💡 The ESP32 runs this web server as another FreeRTOS task alongside your clock and alarm tasks. That's the beauty of the RTOS — everything runs concurrently.

#### 2. Build the HTML/CSS/JS interface

Create a single-page mobile-friendly interface with: a list of current alarms with toggle switches, an "add alarm" button with time picker, day-of-week selection, sound choice, and delete functionality. Embed the HTML as a `const char[]` string in your C code, or store it in SPIFFS (the ESP32's file system).

> 💡 Use the SPIFFS approach — it keeps your C code clean and lets you edit the HTML separately. ESP-IDF has a SPIFFS component and examples.

#### 3. Implement the REST API

Create a simple JSON API: `GET /api/alarms` returns `[{hour: 7, minute: 30, enabled: true, days: [1,1,1,1,1,0,0]}]`. `POST /api/alarms` accepts the same format to update. Use cJSON (included in ESP-IDF) for JSON parsing and generation.

> 💡 cJSON is a lightweight C library for JSON. Learn `cJSON_CreateObject()`, `cJSON_AddNumberToObject()`, and `cJSON_Parse()`. This is the standard way embedded devices handle structured data.

#### 4. Make it discoverable via mDNS

Enable mDNS so you can access your clock at `http://alarmclock.local` instead of remembering an IP address. ESP-IDF has a built-in mDNS component — it takes about 10 lines of code.

> 💡 mDNS works on most phones and computers. On Android you might need to use the IP address instead — mDNS support is inconsistent on Android.

#### 5. Add a "bookmark to home screen" prompt

Add a Web App Manifest (a small JSON file) so that when someone bookmarks the page on iOS or Android, it appears as an app icon on their home screen with a custom name and icon. This makes it feel like a real app.

> 💡 Add `<meta name="apple-mobile-web-app-capable" content="yes">` and a `manifest.json` link in your HTML. Choose a nice alarm clock icon.

### Concepts You'll Learn

- HTTP server fundamentals (requests, responses, status codes)
- REST API design
- JSON serialisation/deserialisation in C (cJSON)
- SPIFFS file system on embedded devices
- mDNS for local network discovery
- Concurrent server: handling web requests while clock runs

---

## Phase 7: Polish & Enclosure

**Weeks 18–20 · ~10–15 hours**

**Goal:** Make it a finished product: handle edge cases, add quality-of-life features, solder it onto a permanent board, and build or buy an enclosure.

### Shopping List

| Item | Price | Where | Note |
|------|-------|-------|------|
| Perfboard / prototype PCB | €2–3 | Amazon.de — search "perfboard prototype PCB" | For soldering a permanent circuit (optional but recommended) |
| Enclosure / case | €0–15 | 3D print (DTU probably has printers), or a small wooden/plastic project box from a hobby shop | Makes it look like a real product on your nightstand |

### Tasks

#### 1. Add display brightness scheduling

Dim the display at night and brighten it during the day. Either use a time-based schedule (dim after 22:00, bright after 07:00) or add a photoresistor (LDR) for automatic ambient brightness sensing.

> 💡 Start with time-based — it's simpler and works well. You can always add the LDR later.

#### 2. Handle all edge cases

Test and handle: Wi-Fi down at boot (show last known time or a blinking display), NTP sync failure, multiple alarms at the same time, power loss during alarm, SD card removed, and timezone changes.

> 💡 Write a comprehensive test plan. Walk through every scenario. This is where hobby projects become reliable products.

#### 3. Solder onto a permanent board

Once your breadboard prototype is stable, solder everything onto a perfboard. Plan the layout first on paper. Keep power traces thick. Add headers for the display and speaker so they can still be disconnected.

> 💡 Take photos of your breadboard wiring before disassembling. Label every wire.

#### 4. Build or print an enclosure

Design a case in Fusion 360 or Onshape and 3D print it (DTU's maker spaces likely have printers). Include cutouts for: the display, speaker grille holes, USB port for power, and button holes. Alternatively, a simple wooden box from a craft shop works great.

> 💡 Measure everything twice. Account for wire clearance inside the case. A friction-fit or snap-fit lid is easier than screws.

#### 5. Add OTA (Over-The-Air) updates

ESP-IDF has built-in OTA support that lets you flash new firmware over Wi-Fi without plugging in a USB cable. This means you can update your alarm clock's code from your computer without touching the clock.

> 💡 This is a stretch goal but incredibly useful. Once it's on your nightstand, you don't want to take it apart every time you improve the code.

### Concepts You'll Learn

- OTA firmware updates
- Robust error handling in embedded systems
- Soldering and PCB layout basics
- Enclosure design and 3D printing/fabrication
- Product thinking: reliability, edge cases, user experience

---

## Complete Shopping List — All Phases

| Item | Price |
|------|-------|
| ESP32 DevKit V1 (USB-C preferred) | €5–10 |
| Breadboard (400 or 830 point) | €3–5 |
| Jumper wires (M-M and M-F mix pack) | €3–5 |
| USB cable (micro-USB or USB-C) | €0–3 |
| TM1637 4-digit 7-segment display module | €2–4 |
| DFPlayer Mini MP3 module | €3–6 |
| Small speaker (3W, 8 Ohm) | €3–5 |
| Micro SD card (8GB) | €3–5 |
| 1kΩ resistor | €0.50 |
| Tactile push buttons (2–3 pcs, 6mm) | €1–2 |
| 10kΩ resistors (2–3 pcs) | €0.50 |
| Perfboard / prototype PCB | €2–3 |
| Enclosure / case | €0–15 |
| **Estimated total** | **€20–45** |

> **Shopping tip:** You only need Phase 1 items to start. Buy components for each phase as you reach it — this way you don't spend money on parts that sit unused for weeks, and you can adjust your plan as you learn.

---

## Recommended EU Shops

- **Amazon.de** — Ships to Denmark, has everything, best for one-stop shopping
- **Electrokit.com** (Sweden) — Quality Nordic electronics shop, fast EU shipping
- **Botland.store** (Poland) — Great prices, ships EU
- **Mouser.dk / DigiKey** — Official parts, slightly pricier but reliable
