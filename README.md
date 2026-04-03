# 🌫️ IoT Air Quality Monitor

> Real-time air quality monitoring using **ESP32 + MQTT + Node-RED + InfluxDB**

![ESP32](https://img.shields.io/badge/ESP32-WiFi%20%2B%20BT-blue?style=for-the-badge&logo=espressif)
![MQTT](https://img.shields.io/badge/MQTT-Mosquitto-orange?style=for-the-badge)
![Node-RED](https://img.shields.io/badge/Node--RED-v3.x-red?style=for-the-badge)
![InfluxDB](https://img.shields.io/badge/InfluxDB-v2.x-purple?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

---

## 📸 Project Preview

| Hardware Setup | Node-RED Flow | Dashboard |
|---|---|---|
| ESP32 + Sensors on breadboard | MQTT → JSON → InfluxDB pipeline | Live gauges & charts |

---

## 📋 Table of Contents

- [Overview](#overview)
- [System Architecture](#system-architecture)
- [Hardware Required](#hardware-required)
- [Wiring Connections](#wiring-connections)
- [Software Requirements](#software-requirements)
- [Installation & Setup](#installation--setup)
- [ESP32 Code Explained](#esp32-code-explained)
- [Node-RED Flow](#node-red-flow)
- [InfluxDB Setup](#influxdb-setup)
- [Dashboard](#dashboard)
- [Project Structure](#project-structure)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## 🔍 Overview

This project monitors indoor air quality in **real-time** using an ESP32 microcontroller connected to:
- **DHT11** — Temperature & Humidity
- **MQ9** — CO (Carbon Monoxide) & LPG Gas
- **MQ135** — Air Quality (NH3, CO2, Benzene, etc.)

Data flows wirelessly over **WiFi via MQTT** to **Node-RED**, which processes and stores it in **InfluxDB**, displayed on a live **Node-RED Dashboard**.

---

## 🏗️ System Architecture

```
┌─────────────┐    WiFi/MQTT    ┌───────────┐    Process    ┌──────────┐
│    ESP32    │ ─────────────▶  │  Node-RED │ ────────────▶│ InfluxDB │
│  + Sensors  │                 │  Flow     │               │ Database │
└─────────────┘                 └───────────┘               └──────────┘
                                      │                           │
                                      ▼                           ▼
                               ┌─────────────┐         ┌──────────────────┐
                               │  Dashboard  │         │  Historical Data  │
                               │  (Live UI)  │         │  Time-series DB  │
                               └─────────────┘         └──────────────────┘

MQTT Topic: esp32/sensors
JSON Format: {"temp":31.8, "hum":35, "mq9_ppm":26.72, "mq135_ppm":0}
```

---

## 🛒 Hardware Required

| Component | Quantity | Notes |
|---|---|---|
| ESP32 DevKit v1 | 1 | Any 30-pin ESP32 board |
| DHT11 Sensor | 1 | Temperature & Humidity |
| MQ9 Gas Sensor | 1 | CO + LPG detection |
| MQ135 Gas Sensor | 1 | Air quality (CO2, NH3) |
| Breadboard | 1 | Full-size recommended |
| Jumper Wires | ~15 | Male-to-Male |
| USB Cable | 1 | Micro-USB for ESP32 |

---

## 🔌 Wiring Connections

### DHT11 → ESP32
| DHT11 Pin | ESP32 Pin |
|---|---|
| VCC | 3.3V |
| GND | GND |
| DATA | GPIO 4 |

### MQ9 → ESP32
| MQ9 Pin | ESP32 Pin |
|---|---|
| VCC | VIN (5V) |
| GND | GND |
| AO (Analog Out) | GPIO 34 |

### MQ135 → ESP32
| MQ135 Pin | ESP32 Pin |
|---|---|
| VCC | VIN (5V) |
| GND | GND |
| AO (Analog Out) | GPIO 35 |

> ⚠️ **Note:** MQ sensors need 5V — use the **VIN pin** (not 3.3V). GPIO 34 and 35 are **input-only** ADC pins, perfect for analog sensors.

---

## 💻 Software Requirements

### On Your PC / Server
| Software | Version | Download |
|---|---|---|
| Arduino IDE | 2.x | [arduino.cc](https://www.arduino.cc/en/software) |
| Mosquitto MQTT Broker | Latest | [mosquitto.org](https://mosquitto.org/download/) |
| Node-RED | 3.x | [nodered.org](https://nodered.org) |
| InfluxDB | v2.x | [influxdata.com](https://www.influxdata.com/downloads/) |
| Node.js | 18+ LTS | [nodejs.org](https://nodejs.org) |

### Arduino Libraries (install via Library Manager)
| Library | Author |
|---|---|
| `PubSubClient` | Nick O'Leary |
| `DHT sensor library` | Adafruit |
| `Adafruit Unified Sensor` | Adafruit |

---

## 🚀 Installation & Setup

### Step 1 — Arduino IDE Setup

1. Open Arduino IDE → File → Preferences
2. Add ESP32 board URL:
   ```
   https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   ```
3. Tools → Board → Board Manager → search **esp32** → Install
4. Install libraries via **Sketch → Include Library → Manage Libraries**

### Step 2 — Mosquitto MQTT Broker

**Windows:**
```
Download installer from https://mosquitto.org/download/
```

**Linux:**
```bash
sudo apt update
sudo apt install mosquitto mosquitto-clients
sudo systemctl start mosquitto
sudo systemctl enable mosquitto
```

**Edit config to allow external connections:**
```bash
# Edit /etc/mosquitto/mosquitto.conf
listener 1883
allow_anonymous true
```

**Test it works:**
```bash
# Terminal 1 - Subscribe
mosquitto_sub -h localhost -t "esp32/sensors"

# Terminal 2 - Publish test
mosquitto_pub -h localhost -t "esp32/sensors" -m '{"temp":31.8,"hum":35,"mq9_ppm":26.72,"mq135_ppm":0}'
```

### Step 3 — InfluxDB Setup

1. Download and install InfluxDB v2 from [influxdata.com](https://www.influxdata.com/downloads/)
2. Start InfluxDB and open `http://localhost:8086`
3. Create initial account, then:
   - **Organization:** `iot_org` (or your choice)
   - **Bucket:** `iot_data`
4. Go to **Data → Tokens** → Generate API Token → Copy it

### Step 4 — Node-RED Setup

```bash
# Install Node-RED globally
npm install -g node-red

# Start Node-RED
node-red
```

Open `http://localhost:1880` in browser.

**Install required palettes:**
- Go to Menu (☰) → Manage Palette → Install
- Search and install:
  - `node-red-dashboard`
  - `node-red-contrib-influxdb`

**Import the flow:**
- Menu → Import → paste contents of `nodered_flow.json`
- Configure MQTT broker node with your PC's IP
- Configure InfluxDB node with your token, org, bucket

### Step 5 — ESP32 Code Upload

1. Open `air_quality_esp32.ino` in Arduino IDE
2. Update these values:
   ```cpp
   const char* ssid     = "YOUR_WIFI_NAME";
   const char* password = "YOUR_WIFI_PASSWORD";
   const char* mqtt_server = "YOUR_PC_IP";  // e.g. 192.168.1.100
   ```
3. Find your PC's IP:
   - **Windows:** Run `ipconfig` in CMD
   - **Linux/Mac:** Run `ifconfig` in terminal
4. Select Board: **ESP32 Dev Module**
5. Select correct COM Port
6. Upload!

---

## 📟 ESP32 Code Explained

```cpp
// PPM Calculation using sensor curve fitting
float getPPM(int adc, float R0, float m, float b) {
  float voltage = (adc / 4095.0) * 3.3;   // ADC to voltage
  float Rs = ((3.3 - voltage) / voltage) * RL;  // Sensor resistance
  float ratio = Rs / R0;                    // Rs/R0 ratio
  float ppm = pow(10, (log10(ratio) - b) / m);  // Log curve fit
  return ppm;
}
```

| Parameter | MQ9 | MQ135 |
|---|---|---|
| `m` (slope) | -0.77 | -0.42 |
| `b` (intercept) | 1.7 | 1.92 |
| Detects | CO, LPG | CO2, NH3, Benzene |

> 💡 **Calibration Tip:** `R0` values (default 10.0 kΩ) should be calibrated in clean fresh air. Read the ADC for 24 hours in fresh air and calculate the actual R0 value.

---

## 🔴 Node-RED Flow

### Flow Structure
```
[MQTT In: esp32/sensors]
        │
        ▼
   [JSON Parse]
        │
        ▼
  [Function Node] ──▶ [Temperature Gauge]
        │         ──▶ [Humidity Gauge]
        │         ──▶ [MQ9_ppm Gauge]
        │         ──▶ [MQ9 Gas Chart]
        │         ──▶ [MQ135_ppm Gauge]
        │         ──▶ [MQ135 Gas Chart]
        │
        ▼
 [Debug Node]
        │
        ▼
 [InfluxDB Write]
```

### Function Node Code
```javascript
let d = msg.payload;
let msg1 = { payload: d.temp };
let msg2 = { payload: d.hum };
let msg3 = { payload: d.mq9_ppm };
let msg4 = { payload: d.mq9_ppm };
let msg5 = { payload: d.mq135_ppm };
let msg6 = { payload: d.mq135_ppm };
return [msg1, msg2, msg3, msg4, msg5, msg6];
```

---

## 🗄️ InfluxDB Setup

### Node Configuration in Node-RED
```
Version      : 2.0
URL          : http://localhost:8086
Token        : <your-api-token>
Organization : iot_org
Bucket       : iot_data
Measurement  : sensor_data
```

### Sample Flux Query (InfluxDB Explorer)
```flux
from(bucket: "iot_data")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "sensor_data")
  |> filter(fn: (r) => r._field == "temp" or r._field == "hum")
```

---

## 📊 Dashboard

Access the live dashboard at: `http://localhost:1880/ui`

| Widget | Type | Range |
|---|---|---|
| Temperature | Gauge | 0–80°C |
| Humidity | Gauge | 0–100% |
| MQ9 PPM | Gauge | 0–1000 ppm |
| MQ9 Gas | Line Chart | Time-series |
| MQ135 PPM | Gauge | 0–1000 ppm |
| MQ135 Gas | Line Chart | Time-series |

---

## 📁 Project Structure

```
iot-air-quality-monitor/
│
├── 📄 README.md                  ← This file
├── 📄 LICENSE
│
├── 📁 arduino/
│   └── 📄 air_quality_esp32.ino  ← ESP32 main code
│
├── 📁 nodered/
│   └── 📄 nodered_flow.json      ← Import this in Node-RED
│
├── 📁 docs/
│   ├── 📄 wiring_diagram.png     ← Circuit diagram
│   └── 📄 dashboard_preview.png  ← Dashboard screenshot
│
└── 📁 images/
    ├── 📄 hardware_setup.jpg
    └── 📄 nodered_flow.jpg
```

---

## 🔧 Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| ESP32 won't connect to WiFi | Wrong credentials | Double-check SSID & password |
| MQTT connection failed | Wrong IP | Run `ipconfig`/`ifconfig` to get correct PC IP |
| DHT reads NaN | Loose wire or wrong pin | Check GPIO 4 wiring |
| MQ sensor gives 0 or very high values | Needs warmup time | Wait 2–5 minutes after powering on |
| Node-RED not receiving data | Mosquitto not running | Run `sudo systemctl status mosquitto` |
| InfluxDB node shows error | Wrong token | Regenerate token in InfluxDB → Data → Tokens |
| Dashboard shows no data | Flow not deployed | Click **Deploy** button in Node-RED |

---

## 📡 MQTT Data Format

**Topic:** `esp32/sensors`

**Payload (JSON):**
```json
{
  "temp": 31.8,
  "hum": 35,
  "mq9_ppm": 26.72,
  "mq135_ppm": 1.02
}
```

---

## 🧪 Gas Level Reference

| Gas | Safe Level | Warning | Dangerous |
|---|---|---|---|
| CO (MQ9) | < 35 ppm | 35–200 ppm | > 200 ppm |
| Air Quality (MQ135) | < 50 ppm | 50–200 ppm | > 200 ppm |
| Humidity | 40–60% | < 30% or > 70% | — |
| Temperature | 20–26°C | — | — |

Author:
Ravi Patel
