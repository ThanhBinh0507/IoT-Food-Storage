# 🌿 FreshGuard: IoT Food Storage Management System - Firmware

This repository contains the **Embedded Firmware** source code for **FreshGuard**, a multi-disciplinary IoT project designed to monitor and control the environment of cold storage facilities for fresh food preservation. 

> **📝 Note on Project Scope:** This project was developed by a cross-functional team of 5 students. **This specific repository focuses solely on the Firmware engineering and IoT hardware communication.** The Web Dashboard (Next.js) and Backend API (Spring Boot) are maintained in separate team repositories.

---

## 👨‍💻 My Role & Responsibilities
As the Firmware Developer for this project, my primary contributions included:
* **MQTT Architecture Design:** Architected the data payload structure and communication flow between the ESP32 node and the Adafruit IO MQTT broker.
* **Alert Mechanism (Debounce):** Implemented a reliable alert system that filters out sensor noise by requiring consecutive out-of-bound readings before triggering a notification.
* **Scheduling & Time Synchronization:** Engineered a schedule mode by synchronizing network time via NTP, adding local timezone offsets, and storing it in the RTC to maintain offline schedule execution.
* **Local UI Integration:** Programmed an I2C LCD1602 to display real-time environmental metrics, connection status, and operational modes directly on the physical node.

---

## 🛠 Hardware Components
Each IoT node in the system represents a specific storage room (e.g., `A1` for Area A, Room 1) and consists of:
* **Microcontroller:** Yolo:bit (ESP32-based platform)
* **Sensors:** 
  * DHT20 (Temperature & Humidity)
  * Light Sensor (Analog)
  * PIR Motion Sensor
* **Actuators:**
  * PWM-controlled Fans (for Temperature and Humidity regulation)
  * RGB LED (Lighting control)
* **Display:** LCD1602 via I2C interface

---

## ⚙️ Firmware Architecture & Key Features

### 1. Cooperative Task Scheduler
Instead of relying on an RTOS, the firmware implements a lightweight **Cooperative Task Scheduler**. The main loop checks elapsed time (`running_time()`) to execute specific tasks at defined intervals without blocking:
* `SensorData()`: Reads sensors and publishes to MQTT every 30 seconds.
* `DeviceControlSignal()`: Publishes device state changes every 4 seconds.
* `LCD & AutoMode()`: Updates the local display and executes Auto logic every 2 seconds.
* `SchedulerMode()`: Reads the RTC and executes time-based schedules every 5 seconds.

### 2. Multi-Mode Operation
The node supports three dynamic operation modes controlled via MQTT:
* **Manual Mode (0):** Direct ON/OFF control of actuators via the web dashboard.
* **Auto Mode (1):** Actuators respond to sensor thresholds. A **Hysteresis control** (dead band) is implemented (e.g., 3°C or 3% humidity) to prevent fans from toggling rapidly when readings fluctuate near the threshold.
* **Schedule Mode (2):** Devices operate based on specific time windows independently (e.g., Fan ON from 06:00 to 18:00).

### 3. Reliable Alerting System
To prevent false alarms from temporary sensor glitches, an alert is only published to the `notification` feed if the temperature or humidity exceeds the threshold for **30 consecutive cycles (~60 seconds)**. Once normal conditions are restored, a safe state (code `0`) is published.

---

## 📡 MQTT Communication Protocol
The system communicates with the Adafruit IO broker using the following optimized feed structure to minimize API limits[cite: 1]:

| Feed | Direction | Payload Example | Description |
| :--- | :--- | :--- | :--- |
| `sensor-data` | Publish | `A1:t=32,h=62,l=81,m=1` | Aggregated data: Temp, Humi, Light, Motion |
| `device-state` | Publish | `A1:t=0,h=1,l=0` | Current state of Temp Fan, Humi Fan, LED |
| `notification` | Publish | `1`, `2`, or `3` | Alert codes for High Temp, High Humi, or Both |
| `mode` | Subscribe | `A1:0`, `A1:1`, `A1:2` | Switches between Manual, Auto, Schedule modes |
| `device-control`| Subscribe | `A1:t=1,h=0,l=1` | Manual commands for actuators |
| `threshold` | Subscribe | `A1:t=35,h=80` | Updates threshold values for Auto mode |
| `schedule` | Subscribe | `A1:t=06:00-18:00,...` | Operating time windows for Schedule mode |

*(Note: Every payload begins with a Room ID prefix like `A1:` so multiple nodes can share the same broker efficiently)*.

---

## 🚀 How to Run
1. Clone this repository.
2. Open the source code in your preferred MicroPython/C++ IDE configured for ESP32.
3. Update the Wi-Fi credentials (`SSID`, `PASSWORD`) and Adafruit IO keys (`AIO_USERNAME`, `AIO_KEY`) in the configuration section.
4. Set the specific `ROOM_ID` (e.g., `A1`) for the node.
5. Flash to the Yolo:bit board.
6. The node will initialize the LCD, connect to Wi-Fi, sync NTP, and begin MQTT operations.

---
*Created by Võ Đặng Thanh Bình as part of a University Multidisciplinary Project.*
