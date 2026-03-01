# Smart Class Building System

Distributed environmental monitoring and edge-based automation system for smart classroom deployment.

---

## 📌 Overview

This project implements a scalable smart classroom system designed for real building environments.

The architecture separates:

- Sensor acquisition layer (STM32 slave nodes)
- Edge control and UI layer (ESP32-S3 master)
- Building dashboard integration (WiFi)
- Mobile monitoring (Flutter app)

The system is designed for reliability, local autonomy, and future scalability across multiple classrooms.

---

## 🏗 Architecture

Master:
- ESP32-S3 (16MB Flash, 8MB PSRAM)
- 3.5” Parallel TFT Touchscreen
- WiFi Dashboard Connectivity
- CAN Bus Master

Slave:
- STM32 Sensor Nodes
- Distributed sensing (4 modules per classroom)

Communication:
- CAN Bus (Differential, terminated)

---

## 🌡 Sensors

Each classroom module includes:

- DHT22 (Temperature & Humidity)
- Human Presence Sensor
- Lux Sensor

Optional:
- CO2 Sensor
- Watt Meter
- UV/Fire Sensor

---

## ⚙ Features

- Real-time environmental monitoring
- Distributed sensing (multi-point)
- Occupancy-based automation
- AC control via IR
- Lighting control via relay
- Local edge decision logic (cloud-independent)
- Debug flag-based firmware tracing

---

## 🧠 Control Logic

- Room occupancy determined via multi-node presence aggregation
- Absence confirmation via time threshold (default 10 minutes)
- Closed-loop temperature validation
- Fail-safe local control even without WiFi

---

## 🛠 Firmware Structure
