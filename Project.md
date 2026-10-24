# College Bus Tracking Using GNSS

A real-time college bus tracking system built on a Reyax **RYS352A** multi-constellation GNSS module, an **Arduino Uno R4 WiFi**, and the **Adafruit IO** cloud platform. The bus position is decoded from NMEA data, published over MQTT every 2 seconds, and displayed live on a map dashboard that can be opened from any web browser.

> **Project Phase-II (404197)** | B.E. Electronics & Telecommunication Engineering | Savitribai Phule Pune University
> Matoshri College of Engineering and Research Centre, Eklahare, Nashik | Academic Year 2025-26

---

## Table of Contents

1. [Background](#1-background)
2. [Problem Statement and Solution](#2-problem-statement-and-solution)
3. [Objectives](#3-objectives)
4. [System Architecture](#4-system-architecture)
5. [Hardware](#5-hardware)
6. [Wiring](#6-wiring)
7. [Software and Libraries](#7-software-and-libraries)
8. [Firmware Explained](#8-firmware-explained)
9. [Setup Guide](#9-setup-guide)
10. [Adafruit IO Configuration](#10-adafruit-io-configuration)
11. [Testing the GNSS Antenna](#11-testing-the-gnss-antenna)
12. [Error Detection and Correction](#12-error-detection-and-correction)
13. [Performance Analysis](#13-performance-analysis)
14. [Results](#14-results)
15. [Advantages and Disadvantages](#15-advantages-and-disadvantages)
16. [Applications](#16-applications)
17. [Future Scope](#17-future-scope)
18. [Bill of Material](#18-bill-of-material)
19. [Team](#19-team)
20. [References](#20-references)

---

## 1. Background

**GNSS (Global Navigation Satellite System)** is the general term for any satellite constellation that provides positioning, navigation and timing (PNT) services anywhere on Earth. Satellites broadcast their position and the exact time. A receiver that sees at least **four** satellites measures the signal travel time to each one, converts it to a distance, and uses trilateration to compute its 3D position (latitude, longitude, altitude).

| System | Operator | Coverage |
|--------|----------|----------|
| GPS | United States | Global, most widely used |
| GLONASS | Russia | Global |
| Galileo | European Union | Global, designed for civilian use |
| BeiDou | China | Global |
| NavIC / IRNSS | India (ISRO) | Regional, India and about 1500 km around it |
| QZSS | Japan | Regional (Asia-Oceania), augments GPS |

Using several constellations together improves accuracy and availability, which is why this project uses a multi-GNSS receiver.

## 2. Problem Statement and Solution

### Problems with traditional college bus systems

- **No real-time information:** students and parents cannot see where the bus is or when it will arrive, leading to long waits and missed classes.
- **Safety and security concerns:** without live tracking it is hard to respond quickly to accidents, breakdowns or medical emergencies.
- **Inefficient route planning:** fixed routes increase travel time, fuel use and vehicle wear.
- **Poor communication:** delays, diversions and cancellations are not passed on effectively.
- **High administrative workload:** monitoring is done by phone calls, registers and paperwork.
- **No performance data:** colleges cannot analyze route efficiency, stoppage time or driver behavior.
- **No automated emergency alerts** to parents or authorities.

### Proposed solution

A GNSS receiver on the bus sends its location to a cloud server, and users view the position on a map dashboard. This provides real-time tracking, a centralized view of the fleet, a base for notifications and alerts, and stored data for reports and route planning.

## 3. Objectives

**Primary**
- Real-time location tracking of college buses using GNSS
- Improved student safety through continuous monitoring
- Higher efficiency through better route and schedule management

**Secondary**
- Automated notifications about bus location, arrival time and delays
- Effective communication between students, parents and administration
- A user-friendly interface for all users

**Functional:** bus location tracking, route optimization, notification system, reporting and analytics.
**Non-functional:** reliability (24/7), security of student data, scalability, usability.

## 4. System Architecture

```
┌────────────────┐   UART    ┌─────────────────────┐  Wi-Fi / MQTT  ┌──────────────┐     ┌────────────────┐
│  RYS352A GNSS  │ ────────► │ Arduino Uno R4 WiFi │ ─────────────► │  Adafruit IO │ ──► │ Map dashboard  │
│  module        │   NMEA    │ (TinyGPS++ decoder) │   CSV payload  │  (feed)      │     │ (web browser)  │
└────────────────┘           └─────────────────────┘                └──────────────┘     └────────────────┘
```

**Antenna field-test path (optional):** RYS352A → FT232RL USB-to-UART converter → PC running VisualGPSView.

**Data flow**
1. The RYS352A tracks satellites and outputs NMEA 0183 sentences at 115200 baud.
2. The Arduino reads the serial stream and feeds each byte to TinyGPS++.
3. Every 2 seconds, if the location is valid, the Arduino builds a CSV string and publishes it to the Adafruit IO feed.
4. The Adafruit IO map block plots the received coordinates.

## 5. Hardware

### 5.1 Reyax RYS352A GNSS module

A +3.3 V UART GNSS antenna module with an embedded GPS/GLONASS/BeiDou antenna.

| Parameter | Value |
|-----------|-------|
| GNSS engine | Airoha (MTK) AG3352 |
| Constellations | GPS, GLONASS, Galileo, BeiDou, QZSS (plus SBAS, WAAS, EGNOS, MSAS, GAGAN, AGNSS) |
| Protocol | NMEA 0183 v4.10 |
| Supply voltage | 2.5 V min, 3.3 V typical, 3.63 V max |
| Default baud rate | 115200 (8-N-1) |
| Tracking channels | 47 |
| Navigation update rate | 1 Hz typical, 10 Hz max |
| Accuracy | 1.5 m CEP50 (24 h static, more than 6 satellites, -130 dBm) |
| Tracking sensitivity | -167 dBm |
| Cold start | 26 s without AGNSS, 12 s with EASY |
| Warm / Hot start | 20 s / 1 s without AGNSS |
| Current | about 33 mA acquisition and tracking (passive antenna) |
| 1PPS accuracy | 100 ns |
| Max velocity | 500 m/s |
| Operating temperature | -40 °C to +85 °C |
| Weight | 13.6 g |
| Extras | RTC battery backup, LNA, SAW filter, TCXO, 12 multi-tone interference cancellers, PPS and TXD LEDs |

**Pin description (6-pin Molex 53261-0671 connector)**

| Pin | Name | Direction | Function |
|-----|------|-----------|----------|
| 1 | 1PPS | Output | Time pulse output |
| 2 | GND | - | Ground |
| 3 | TXD | Output | Serial output |
| 4 | RXD | Input | Serial input |
| 5 | VCC | Input | Power and I/O voltage (2.5 to 3.63 V) |
| 6 | RESET_N | Input | Pull low for at least 100 ms to reset |

The board also exposes 12 through-hole pins (pins 7 to 12 are all ground).

### 5.2 Arduino Uno R4 WiFi

| Parameter | Value |
|-----------|-------|
| Microcontroller | Renesas RA4M1 (Arm Cortex-M4, 48 MHz, 32-bit) |
| Wi-Fi / Bluetooth | Espressif ESP32-S3-MINI-1-N8 |
| Operating voltage | 5 V (the ESP32-S3 side is 3.3 V, with a TXB0108 level translator in between) |
| Flash / SRAM / EEPROM | 256 KB / 32 KB / 8 KB |
| I/O | 14 digital pins, 6 analog inputs |
| Connectivity | USB-C, Qwiic (I2C, 3.3 V only), 12x8 LED matrix |

### 5.3 FT232RL USB to UART converter

Converts USB to serial TX/RX, selectable 3.3 V or 5 V. Used only to connect the GNSS module to a PC for antenna testing.

## 6. Wiring

| RYS352A pin | Arduino Uno R4 WiFi |
|-------------|---------------------|
| VCC (pin 5) | 3.3V |
| GND (pin 2) | GND |
| TXD (pin 3) | D0 (`RxPin`) |
| RXD (pin 4) | D1 (`TxPin`) |

> Power the module from **3.3 V only**. Its digital input is rated up to VCC + 0.3 V, so consider a level shifter or a resistor divider on the Arduino D1 (5 V) to GNSS RXD line for long-term use.

## 7. Software and Libraries

- **Arduino IDE** with the **Arduino UNO R4 Boards** package (provides `WiFiS3`)
- `Adafruit MQTT Library` (`Adafruit_MQTT.h`, `Adafruit_MQTT_Client.h`)
- `TinyGPSPlus` (`TinyGPS++.h`)
- `MegunoLink`
- `ArduinoTimer`
- `SoftwareSerial`
- **Adafruit IO** (cloud service and dashboard)
- **VisualGPSView** (Windows freeware, antenna testing only)

The original sketch also includes `TFT_eSPI.h` and `TFT_eWidget.h`. They are not used by the tracking logic and can be removed if they cause compile errors.

## 8. Firmware Explained

| Part | What it does |
|------|--------------|
| `SoftwareSerial GPSSerial(RxPin, TxPin)` | Serial link to the GNSS module on D0/D1 |
| `TinyGPSPlus GPSDecoder` | Parses NMEA sentences into location, speed and altitude |
| `ArduinoTimer ReportTimer` | Triggers a report every 2 seconds |
| `WiFi.begin(ssid, pass)` | Connects to Wi-Fi, retrying every 10 seconds until connected |
| `MQTT_connect()` | Connects to `io.adafruit.com:1883`, retries 3 times at 5 second intervals |
| `Adafruit_MQTT_Publish gpslatlng` | Publishes to `<username>/feeds/gpslatlng/csv` |

**Main loop**
1. Read any available GNSS byte and pass it to `GPSDecoder.encode()`.
2. Every 2 seconds, check `GPSDecoder.location.isValid()`.
3. If valid, build the string `speed(mph),latitude,longitude,altitude(ft)` using `dtostrf()` and publish it.
4. Print the coordinates to the Serial Monitor. If no fix, print `Location not available`.

**Payload format**

```
speed_mph,latitude,longitude,altitude_ft
```

Adafruit IO's map block reads this CSV as speed, latitude, longitude and elevation.

## 9. Setup Guide

1. **Wire** the module to the Arduino as shown in [Wiring](#6-wiring).
2. **Install** the Arduino IDE, the *Arduino UNO R4 Boards* package, and the libraries listed above.
3. **Set up Adafruit IO** (see next section) and copy your username and AIO key.
4. **Create `arduino_secrets.h`** next to the sketch:
   ```cpp
   #define SECRET_SSID "your-wifi-name"
   #define SECRET_PASS "your-wifi-password"
   ```
5. **Set your account** in the sketch:
   ```cpp
   #define AIO_USERNAME "your_adafruit_username"
   #define AIO_KEY      "your_adafruit_io_key"
   ```
6. **Select** *Arduino UNO R4 WiFi* and the COM port, then compile and upload.
7. **Open the Serial Monitor** at 115200 baud.
8. **Go outdoors** with a clear view of the sky and wait for a fix. When you see `MQTT Connected!` and coordinate lines, check the dashboard.

> Use a **2.4 GHz** Wi-Fi network or mobile hotspot. Never commit your AIO key or Wi-Fi password to a public repository; keep them in `arduino_secrets.h` and add it to `.gitignore`.

## 10. Adafruit IO Configuration

1. Create a free account at [io.adafruit.com](https://io.adafruit.com).
2. **Feeds:** create a feed whose key matches the code (`gpslatlng`). Feeds act as data holders between the device and the dashboard.
3. **Dashboard:** create a dashboard (for example `Reyax_GNSS`) and add a **Map** block.
4. In the block settings, select the `gpslatlng` feed under *Connect Feeds*.
5. Your AIO key is available from the key icon on the IO page.

The Basic (free) plan allows 10 feeds, 5 dashboards and a limited data rate, which is enough for this prototype.

## 11. Testing the GNSS Antenna

Use VisualGPSView to confirm the module works before involving the Arduino:

1. Connect the RYS352A to the PC through the FT232RL converter (3.3 V mode).
2. Download VisualGPSView from [visualgps.net](https://www.visualgps.net/#visualgpsview-content).
3. Click **Tools → Connect GPS**, select the COM port, set baud rate **115200**, click OK.
4. Inspect:
   - **Front Panel Status:** satellites tracked and signal strength (SNR)
   - **Scatter Plot:** position samples and horizontal error
   - **Position Plot:** latitude, longitude and altitude over time
   - **NMEA Monitor:** raw sentences (GPGGA, GPGSA, GPGSV and so on)

This step only checks the antenna and is not part of the final system.

## 12. Error Detection and Correction

| Issue | Cause / Observation | Solution |
|-------|---------------------|----------|
| Code compiles and uploads (about 31% flash, 26% RAM used) but the board resets | Baud rate mismatch in the Serial Monitor | Change the Serial Monitor baud rate until it matches (115200) |
| Board joins the hotspot but MQTT reports **Not authorized to connect** | Authorization with the Adafruit IO account failed and the hotspot was on 5 GHz | Created a new Adafruit IO account with matching credentials and switched the hotspot from 5 GHz to **2.4 GHz** |
| No data on the map | Feed name mismatch | Create the feed with the exact key used in the code and connect it to the dashboard's map block |
| `Location not available` | No satellite fix | Move outdoors and wait for the cold start to complete |

## 13. Performance Analysis

| Parameter | Observation |
|-----------|-------------|
| **Location accuracy** | About 5 to 10 m typical in open areas, better with multi-GNSS. Reduced by buildings, trees, tunnels and multipath |
| **Response time** | Updates every few seconds (2 s in this firmware). The module outputs data continuously and the Wi-Fi link adds a small delay |
| **Signal strength** | At least 4 satellites needed. Quality measured by SNR. Best in open sky |
| **Data transmission** | Latitude, longitude, speed and altitude sent regularly. Poor connectivity can cause delays or temporary data loss |
| **Real-time capability** | Live movement visible on the web dashboard |
| **Speed measurement** | Derived from position changes. Reliable in normal signal conditions, affected by sudden signal loss |
| **Reliability** | Depends on hardware quality, stable power, antenna placement and software |
| **Power consumption** | GNSS module is low-power (about 33 mA). Wi-Fi/GSM transmission draws the most |
| **Scalability** | Cloud platform can handle multiple buses and extra sensors |
| **Security** | Risks include GPS spoofing, network attacks and unauthorized tracking. Mitigate with authentication and encryption |

## 14. Results

- The system tracked the bus position and transmitted it to the cloud dashboard.
- Measured positioning accuracy was about **3 to 5 m in open areas** and **10 to 15 m in urban or obstructed areas**.
- Data was sent at regular intervals, allowing continuous live tracking.
- The communication link was stable for most of testing, with minor delays where network coverage was weak. Transmission resumed automatically once the connection returned.
- The prototype shows GNSS tracking is a practical, low-cost and low-maintenance option for colleges and schools.

## 15. Advantages and Disadvantages

**Advantages**
- Real-time tracking for students, parents and administrators
- Improved safety and faster emergency response
- Less waiting time and better time management
- Route optimization, with fuel and cost savings
- Reduced administrative paperwork
- Data storage for analysis and reports
- Low-cost hardware (about INR 5,000)

**Disadvantages**
- Initial hardware and ongoing data/maintenance costs
- Weak or lost signal in tunnels, dense areas and bad weather
- Dependence on power and internet connectivity
- Privacy and data security concerns
- Requires some technical knowledge for installation and troubleshooting
- Limited accuracy in obstructed areas

## 16. Applications

- Real-time bus monitoring and student safety management
- Arrival and departure notifications
- Route planning and optimization
- Attendance tracking (with RFID or biometric integration)
- Emergency response and traffic/delay management
- Maintenance scheduling, data analysis and reporting
- Parental access to live bus status
- Fleet tracking for school buses, taxis, ambulances and public transport

## 17. Future Scope

- Android and iOS mobile app
- Cellular (GSM/4G) connectivity so tracking works without Wi-Fi
- ETA prediction, geofencing and route-deviation alerts
- AI-based route optimization from historical and traffic data
- Driver behavior monitoring (speed, harsh braking)
- Cloud storage and analytics, predictive maintenance
- RFID or biometric attendance, emergency button, voice alerts
- Multi-bus fleet management
- Filtering algorithms and a high-gain external antenna for better accuracy

## 18. Bill of Material

| Sr. | Component | Model | Qty | Cost (INR) |
|-----|-----------|-------|-----|-----------|
| 1 | GNSS antenna | RYS352A | 1 | 1599 |
| 2 | Arduino Uno R4 WiFi | ABX00087 | 1 | 3000 |
| 3 | USB to UART converter | FT232RL | 1 | 395 |
| 4 | Jumper wires | - | 4 | 4 |
| | **Total** | | | **4998** |

## 19. Team

| Name | Roll No. |
|------|----------|
| Sandip Somnath Barhe | B400580128 |
| Piyush Sanjay Bhadange | B400580129 |
| Vivek Ramesh Bhalerao | B400580131 |

- **Guide:** Mr. P. A. Nawale
- **Head of Department:** Dr. Jayant J. Chopade
- **Director:** Dr. Gajanan K. Kharate

## 20. References

1. H. D. Pham, M. Drieberg, C. C. Nguyen, "Development of Vehicle Tracking System Using GPS and GSM Modem," IEEE ICOS, pp. 89-94, 2013.
2. S. Lee, G. Tewolde, J. Kwon, "Design and Implementation of Vehicle Tracking System Using GPS/GSM/GPRS Technology and Smartphone Application," IEEE WF-IoT, pp. 353-358, 2014.
3. K. Banshiwal et al., "A Review on GSM and GPS Based Vehicle Tracking System," IJRTI, 2023.
4. S. Suryawanshi, V. Sahare, "A Survey of Vehicle Position Prediction System Based on GSM and ANN Structure," IJERT, vol. 4, no. 2, 2015.
5. I. Skog, P. Händel, "In-Car Positioning and Navigation Technologies: A Survey," IEEE T-ITS, vol. 10, no. 1, pp. 4-21, 2009.
6. V. Baby Shalini, "GPS and IoT Based Vehicle Tracking System," Springer LNNS, 2022.
7. R. N. H. Alhwety, N. Elfadil, "Vehicle Tracking System Approaches: A Systematic Literature Review," IJCSMC, vol. 13, no. 8, 2024.
8. X. Shan, A. Cabani, H. Chafouk, "A Survey of Vehicle Localization: Performance Analysis and Challenges," IEEE Access, vol. 11, pp. 107085-107107, 2023.
9. N. Joubert, T. G. Reid, F. Noble, "Developments in Modern GNSS and Its Impact on Autonomous Vehicle Architectures," 2020.
10. A. Mustafa et al., "Vehicle Intrusion and Theft Control System Using GSM and GPS," 2020.
