# Node-Rover — ESP32 Autonomous Environmental Telemetry & Edge Access System

> **Embedded C++ firmware for an ESP32-powered environmental inspection rover with localized web HUD, ultrasonic collision ranging, DHT11 telemetry, and SPI-based RFID hardware access control.**

[![Hardware](https://img.shields.io/badge/Hardware-ESP32%20NodeMCU-red?style=flat-square)](#)
[![Firmware](https://img.shields.io/badge/Language-C%2B%2B%20%2F%20Arduino-blue?style=flat-square&logo=c%2B%2B)](#)
[![Web](https://img.shields.io/badge/Interface-Leaflet.js%20HUD-green?style=flat-square)](#)
[![Bus](https://img.shields.io/badge/Protocols-SPI%20%7C%201--Wire%20%7C%20GPIO-orange?style=flat-square)](#)

---

## 1. System Overview

**Node-Rover** is an autonomous edge computing and environmental monitoring rover engineered for hazardous, industrial, or network-constrained environments. Operating without reliance on external cloud servers, the rover hosts an internal asynchronous web server that delivers real-time telemetry, geographic coordinate visualization, and physical access control directly over local WiFi.

---

## 2. Hardware Architecture & Pinout Mapping

The system is powered by an **ESP32 NodeMCU-32S** dual-core microcontroller. Due to shared internal buses on the ESP32, pins were carefully selected to prevent conflicts between the high-speed SPI bus, analog-to-digital converters, and capacitive touch lines.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                          ESP32 PIN CONNECTION MATRIX                        │
├───────────────┬──────────────┬───────────────┬──────────────────────────────┤
│ COMPONENT     │ BUS TYPE     │ ESP32 GPIO    │ ELECTRICAL NOTES             │
├───────────────┼──────────────┼───────────────┼──────────────────────────────┤
│ MFRC522 RFID  │ SPI          │ SCK: GPIO 18  │ 3.3V Logic Level             │
│               │              │ MISO: GPIO 19 │ Hardware SPI (VSPI)          │
│               │              │ MOSI: GPIO 23 │ Hardware SPI (VSPI)          │
│               │              │ SS/SDA: GPIO 5│ Chip Select                  │
│               │              │ RST: GPIO 22  │ Hardware Reset               │
├───────────────┼──────────────┼───────────────┼──────────────────────────────┤
│ HC-SR04       │ Digital GPIO │ TRIG: GPIO 13 │ Pulse trigger (10us)         │
│ Ultrasonic    │              │ ECHO: GPIO 12 │ Resistor divider 5V -> 3.3V  │
├───────────────┼──────────────┼───────────────┼──────────────────────────────┤
│ DHT11         │ 1-Wire Bus   │ DATA: GPIO 4  │ 10k pull-up resistor to 3.3V │
├───────────────┼──────────────┼───────────────┼──────────────────────────────┤
│ Flame Sensor  │ Analog ADC   │ A0: GPIO 34   │ ADC1 input (WiFi-safe)       │
├───────────────┼──────────────┼───────────────┼──────────────────────────────┤
│ Piezo Buzzer  │ PWM / GPIO   │ SIG: GPIO 2   │ Active high alarm buzzer     │
└───────────────┴──────────────┴───────────────┴──────────────────────────────┘
```

> **Critical Safety Note on GPIO 34:** Pin 34 belongs to **ADC1**. Unlike ADC2 pins, ADC1 can be read continuously while the ESP32 WiFi radio is actively transmitting packets.

---

## 3. Firmware Architecture (`code.ino`)

The firmware is structured around a **non-blocking state machine** using software timers (`millis()`), completely avoiding the `delay()` function which causes watchdog timer resets and drops incoming network packets.

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                       NON-BLOCKING EXECUTION CYCLE                          │
└─────────────────────────────────────────────────────────────────────────────┘

                  ┌───────────────────────────────┐
                  │          void loop()          │
                  └──────────────┬────────────────┘
                                 │
         ┌───────────────────────┼───────────────────────┐
         ▼                       ▼                       ▼
  [ RFID Polling ]       [ Timer: 200ms ]        [ Timer: 2000ms ]
  SPI scan for token     Ultrasonic Ranging      DHT11 Poll (Slow)
  Auth Token -> Unlock   Distance < 15cm -> Stop Update JSON struct
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 ▼
                     [ AsyncWebServer Task ]
                     Serves Leaflet.js HUD
                     Broadcasts /telemetry JSON
```

### Key Technical Implementations:
1. **PROGMEM Web Server Storage:** The complete HTML, CSS, and Leaflet.js client application is stored in the ESP32’s flash memory via `PROGMEM`, saving SRAM for dynamic buffer allocations.
2. **Deterministic Distance Calculation:**
   $$	ext{Distance (cm)} = rac{	ext{Duration (\mu s)} 	imes 0.0343}{2}$$
3. **Safety Interlock:** If the analog flame sensor detects a value exceeding the threshold or ultrasonic distance drops below 15 cm, the rover immediately enters a safety lockout state, sounding the buzzer and cutting motor output.

---

## 4. Software Dependencies
- `ESPAsyncWebServer` & `AsyncTCP`
- `MFRC522` (RFID Library by Miguel Balboa)
- `DHT sensor library` (Adafruit)
- `ArduinoJson` (v6.x)

---

## 5. Deployment & Flashing Instructions

1. Open `code.ino` in the **Arduino IDE** or **PlatformIO**.
2. Install the ESP32 board package (`Tools > Board > Board Manager > esp32 by Espressif`).
3. Set board to `ESP32 Dev Module`.
4. Configure flash frequency to `80MHz` and upload speed to `921600`.
5. Connect ESP32 via Micro-USB, press and hold `BOOT` during flashing if required.
6. Connect your laptop/phone to the `Node-Rover-AP` WiFi network and open `http://192.168.4.1` to access the live Leaflet telemetry HUD.
