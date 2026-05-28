# 🌿 FreshGuard: IoT Food Storage Management System - Firmware

This repository contains the **Embedded Firmware** source code for **FreshGuard**, a multi-disciplinary IoT project designed to monitor and control the environment of cold storage facilities for fresh food preservation[cite: 1]. 

> **📝 Note on Project Scope:** This project was developed by a cross-functional team of 5 students[cite: 1]. **This specific repository focuses solely on the Firmware engineering and IoT hardware communication.** The Web Dashboard (Next.js) and Backend API (Spring Boot) are maintained in separate team repositories[cite: 1].

---

## 👨‍💻 My Role & Responsibilities
As the Firmware Developer for this project, my primary contributions included:
* **MQTT Architecture Design:** Architected the data payload structure and communication flow between the ESP32 node and the Adafruit IO MQTT broker[cite: 1].
* **Alert Mechanism (Debounce):** Implemented a reliable alert system that filters out sensor noise by requiring consecutive out-of-bound readings before triggering a notification[cite: 1].
* **Scheduling & Time Synchronization:** Engineered a schedule mode by synchronizing network time via NTP, adding local timezone offsets, and storing it in the RTC to maintain offline schedule execution[cite: 1].
* **Local UI Integration:** Programmed an I2C LCD1602 to display real-time environmental metrics, connection status, and operational modes directly on the physical node[cite: 1].

---

## 🛠 Hardware Components
Each IoT node in the system represents a specific storage room (e.g., `A1` for Area A, Room 1) and consists of[cite: 1]:
* **Microcontroller:** Yolo:bit (ESP32-based platform)[cite: 1]
* **Sensors:** 
  * DHT20 (Temperature & Humidity)[cite: 1]
  * Light Sensor (Analog)[cite: 1]
  * PIR Motion Sensor[cite: 1]
* **Actuators:**
  * PWM-controlled Fans (for Temperature and Humidity regulation)[cite: 1]
  * RGB LED (Lighting control)[cite: 1]
* **Display:** LCD1602 via I2C interface[cite: 1]

---

## ⚙️ Firmware Architecture & Key Features

### 1. Cooperative Task Scheduler
Instead of relying on an RTOS, the firmware implements a lightweight **Cooperative Task Scheduler**[cite: 1]. The main loop checks elapsed time (`running_time()`) to execute specific tasks at defined intervals without blocking[cite: 1]:
* `SensorData()`: Reads sensors and publishes to MQTT every 30 seconds[cite: 1].
* `DeviceControlSignal()`: Publishes device state changes every 4 seconds[cite: 1].
* `LCD & AutoMode()`: Updates the local display and executes Auto logic every 2 seconds[cite: 1].
* `SchedulerMode()`: Reads the RTC and executes time-based schedules every 5 seconds[cite: 1].

### 2. Multi-Mode Operation
The node supports three dynamic operation modes controlled via MQTT[cite: 1]:
* **Manual Mode (0):** Direct ON/OFF control of actuators via the web dashboard[cite: 1].
* **Auto Mode (1):** Actuators respond to sensor thresholds. A **Hysteresis control** (dead band) is implemented (e.g., 3°C or 3% humidity) to prevent fans from toggling rapidly when readings fluctuate near the threshold[cite: 1].
* **Schedule Mode (2):** Devices operate based on specific time windows independently (e.g., Fan ON from 06:00 to 18:00)[cite: 1].

### 3. Reliable Alerting System
To prevent false alarms from temporary sensor glitches, an alert is only published to the `notification` feed if the temperature or humidity exceeds the threshold for **30 consecutive cycles (~60 seconds)**[cite: 1]. Once normal conditions are restored, a safe state (code `0`) is published[cite: 1].

---

## 📡 MQTT Communication Protocol
The system communicates with the Adafruit IO broker using the following optimized feed structure to minimize API limits[cite: 1]:

| Feed | Direction | Payload Example | Description |
| :--- | :--- | :--- | :--- |
| `sensor-data` | Publish | `A1:t=32,h=62,l=81,m=1` | Aggregated data: Temp, Humi, Light, Motion[cite: 1] |
| `device-state` | Publish | `A1:t=0,h=1,l=0` | Current state of Temp Fan, Humi Fan, LED[cite: 1] |
| `notification` | Publish | `1`, `2`, or `3` | Alert codes for High Temp, High Humi, or Both[cite: 1] |
| `mode` | Subscribe | `A1:0`, `A1:1`, `A1:2` | Switches between Manual, Auto, Schedule modes[cite: 1] |
| `device-control`| Subscribe | `A1:t=1,h=0,l=1` | Manual commands for actuators[cite: 1] |
| `threshold` | Subscribe | `A1:t=35,h=80` | Updates threshold values for Auto mode[cite: 1] |
| `schedule` | Subscribe | `A1:t=06:00-18:00,...` | Operating time windows for Schedule mode[cite: 1] |

*(Note: Every payload begins with a Room ID prefix like `A1:` so multiple nodes can share the same broker efficiently)*[cite: 1].

---

## 🚀 How to Run
1. Clone this repository.
2. Open the source code in your preferred MicroPython/C++ IDE configured for ESP32.
3. Update the Wi-Fi credentials (`SSID`, `PASSWORD`) and Adafruit IO keys (`AIO_USERNAME`, `AIO_KEY`) in the configuration section.
4. Set the specific `ROOM_ID` (e.g., `A1`) for the node.
5. Flash to the Yolo:bit board.
6. The node will initialize the LCD, connect to Wi-Fi, sync NTP, and begin MQTT operations[cite: 1].

---
*Created by Võ Đặng Thanh Bình as part of a University Multidisciplinary Project.*
