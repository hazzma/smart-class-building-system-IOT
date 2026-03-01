# SMART CLASS SYSTEM

## Functional Specification Document (FSD)

---

# 0. Firmware Architecture Guideline

FRAMEWORK POLICY

Master MCU:

* ESP32-S3 (16MB Flash + 8MB PSRAM)

Slave MCU:

* STM32 (per sensor module)

Framework:

* Arduino (PlatformIO) for rapid development
* Architecture SHALL remain portable to ESP-IDF (Master)

ARCHITECTURE RULES

RULE-001: Master and Slave responsibilities MUST be strictly separated.
RULE-002: Slave firmware SHALL only handle sensing and low-level acquisition.
RULE-003: Master firmware SHALL handle UI, networking, logging, and control logic.
RULE-004: No blocking delay() allowed in communication layer.
RULE-005: All inter-MCU communication SHALL use structured packet protocol.
RULE-006: System SHALL remain operational even if WiFi is disconnected.
RULE-007: Control decisions SHALL NOT depend on cloud availability.

---

# 1. Document Information

| Field         | Value                                   |
| ------------- | --------------------------------------- |
| Version       | 0.1                                     |
| Status        | Draft                                   |
| Master MCU    | ESP32-S3                                |
| Slave MCU     | STM32                                   |
| Display       | 3.5" Parallel TFT Touch                 |
| Communication | CAN Bus (planned)                       |
| Application   | Smart Classroom Monitoring & Automation |

---

# 2. System Overview

## 2.1 Purpose

This system provides environmental monitoring and automated control for classroom environments.

Primary Goals:

* Real-time monitoring by Building Management
* Automatic energy optimization when room is unoccupied
* Scalable architecture for multi-classroom deployment

---

# 3. System Context Diagram

```
                ┌────────────────────────┐
                │   Building Dashboard   │
                │     (Web / Local)      │
                └───────────┬────────────┘
                            │ WiFi
                            ▼
                   ┌───────────────────┐
                   │   ESP32-S3 Master │
                   │ UI + Control      │
                   └────────┬──────────┘
                            │ CAN Bus
     ┌──────────────────────┼──────────────────────┐
     ▼                      ▼                      ▼
┌───────────┐        ┌───────────┐        ┌───────────┐
│ STM32 #1  │        │ STM32 #2  │        │ STM32 #3  │
│ SensorMod │        │ SensorMod │        │ SensorMod │
└───────────┘        └───────────┘        └───────────┘
```

---

# 4. Sensor Architecture

Each classroom contains 4 distributed sensor modules.

Each module contains:

* DHT22 (Temperature & Humidity)
* Human Presence Sensor
* Lux Sensor

Optional centralized sensors:

* CO2 sensor
* Watt Meter
* UV/Fire Sensor

SENS-001: Distributed sensing SHALL be used to avoid single-point measurement bias.
SENS-002: Master SHALL aggregate and average data across modules.
SENS-003: Presence validation SHALL require time-based confirmation.

---

# 5. Communication Architecture (CAN Bus)

COMM-001: CAN Bus SHALL be differential (CAN_H, CAN_L).
COMM-002: 120Ω termination SHALL exist at both ends of bus.
COMM-003: All nodes SHALL use unique Node ID.
COMM-004: Message priority SHALL be based on CAN ID.

## 5.1 Message Structure

Example ID Allocation:

| CAN ID | Function             |
| ------ | -------------------- |
| 0x101  | Module 1 Sensor Data |
| 0x102  | Module 2 Sensor Data |
| 0x200  | CO2 Data             |
| 0x300  | Master Command       |

Packet Payload Example (8 bytes max):

Byte 0-1 : Temperature (int16 scaled)
Byte 2-3 : Humidity
Byte 4   : Presence
Byte 5-6 : Lux
Byte 7   : Status Flags

---

# 6. Control Logic

## 6.1 Occupancy Logic

CTRL-001: Room SHALL be considered occupied if any module detects presence within time window T.
CTRL-002: Default absence confirmation time SHALL be 10 minutes.
CTRL-003: CO2 trend MAY be used as secondary validation.

## 6.2 Automatic Actions

If unoccupied:

* Turn off Lights
* Set AC to Eco Mode or OFF

If occupied:

* Restore previous state

CTRL-010: Control actions SHALL use closed-loop validation via temperature feedback.

---

# 7. Output Interfaces

* IR LED for AC & Projector
* Relay Module for Lighting Control

SAFE-001: Relay outputs SHALL use opto-isolation.
SAFE-002: AC control SHALL not depend solely on IR acknowledgment.
SAFE-003: System SHALL verify environmental response after command.

---

# 8. Debug & Diagnostic Flags

System SHALL include compile-time debug flags.

Example:

```cpp
#define DEBUG_SERIAL
#define DEBUG_CAN
#define DEBUG_SENSOR
#define DEBUG_CONTROL
```

DBG-001: DEBUG_SERIAL enables UART logging.
DBG-002: DEBUG_CAN logs frame transmission and reception.
DBG-003: DEBUG_SENSOR logs raw sensor values.
DBG-004: DEBUG_CONTROL logs decision tree execution.

All debug prints MUST be wrapped:

```cpp
#ifdef DEBUG_CAN
Serial.println("CAN Frame Sent");
#endif
```

DBG-010: Debug logs SHALL be removable at compile time.
DBG-011: No debug print allowed inside timing-critical interrupt.

---

# 9. Fail-Safe Requirements

FAIL-001: If Master fails, Slave SHALL continue sensing.
FAIL-002: If CAN Bus fails, Master SHALL enter safe mode.
FAIL-003: If WiFi fails, local control SHALL remain active.
FAIL-004: AC default state SHALL not oscillate due to transient presence.

---

# 10. Firmware Structure

```
smart-class/
│
├── master/
│   ├── main.cpp
│   ├── can_manager.cpp
│   ├── control_logic.cpp
│   ├── ui_manager.cpp
│   └── debug_flags.h
│
├── slave/
│   ├── main.cpp
│   ├── sensor_driver.cpp
│   ├── can_node.cpp
│
└── docs/
    └── FSD.md
```

ARCH-001: Control logic MUST be centralized in Master.
ARCH-002: Slave SHALL NOT implement automation logic.
ARCH-003: UI layer SHALL NOT access raw GPIO.
ARCH-004: Communication layer SHALL be isolated.

---

# 11. Non-Functional Requirements

| Metric                   | Requirement               |
| ------------------------ | ------------------------- |
| CAN latency              | < 10 ms per frame         |
| Sampling rate            | Configurable (default 2s) |
| Occupancy decision delay | 10 min default            |
| System uptime            | 24/7 capable              |

---

# 12. Future Extensions

* Digital Twin integration
* Predictive AC control
* Energy per student metric
* Multi-floor aggregation

---

# 13. Revision History

| Version | Date       | Changes                       |
| ------- | ---------- | ----------------------------- |
| 0.1     | 2026-03-01 | Initial Smart Class FSD draft |
