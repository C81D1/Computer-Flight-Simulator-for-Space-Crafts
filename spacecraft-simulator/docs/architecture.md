# Spacecraft Simulator — Component Architecture

## Overview

This document describes the component architecture of the Spacecraft Simulator. It explains not just what each component does, but **what data flows through it and what happens to that data**.

The key architectural principle is:

> **Sensors provide data → subsystems process/model the data → Flight Computer makes decisions → Fault Manager identifies problems → State Machine determines the spacecraft's operational state → Telemetry records everything.**

This gives a realistic **flight-software architecture** rather than simply a collection of unrelated C++ classes.

---

## Component Table

| Component | Data Used | How the Data Is Used | Example |
| --- | --- | --- | --- |
| **Sensors** | Temperature, battery voltage, altitude, velocity, gyroscope readings | Generates simulated measurements representing what real spacecraft sensors would report. Can introduce noise or sensor failures. | Temperature sensor reports **72°C**, altitude reports **400 km**, battery reports **38%**. |
| **Flight Computer** | Sensor measurements, subsystem status, faults, current spacecraft state | Acts as the main decision-making system. Evaluates incoming data and determines what the spacecraft should do. | Receives **battery = 12%** → determines that the spacecraft is in a low-power condition → requests **SAFE_MODE**. |
| **Power System** | Battery level, solar generation, power consumption, subsystem power usage | Simulates how electrical power is generated, stored, and consumed. Calculates whether enough power is available. | Solar panels generate **850 W**, spacecraft consumes **700 W** → battery begins charging. |
| **Thermal System** | Temperature, heat generation, heater status, cooling status | Models spacecraft temperature and determines whether components are operating within safe thermal limits. | Temperature reaches **85°C** → thermal system reports an overheating condition. |
| **Communications** | Ground commands, telemetry packets, connection status, signal/link status | Simulates the communication link between the spacecraft and ground station. Sends spacecraft data and receives commands. | Ground sends **ENTER_SAFE_MODE** → communications passes the command to the Flight Computer. |
| **Fault Manager** | Sensor readings, power status, temperature, communications status, CPU load | Checks system data against predefined safety limits and identifies abnormal conditions. It converts abnormal conditions into fault events. | Battery drops below **15%** → generates `LOW_BATTERY` fault. |
| **State Machine** | Current state, fault events, recovery conditions, commands | Controls the spacecraft's operational mode and determines when transitions between states should occur. | `NOMINAL` + `LOW_BATTERY` → `SAFE_MODE`. |
| **Telemetry** | Sensor readings, spacecraft state, faults, power, thermal and communication data | Collects and timestamps important system information and stores it for monitoring and later analysis. | Records `12:03:15, NOMINAL, Battery=74%, Temp=42°C, Altitude=400km`. |
| **Tests** | Simulated inputs, expected outputs, fault conditions | Verifies that individual components and the complete system behave correctly under normal and abnormal conditions. | Inject `LOW_BATTERY` → test expects spacecraft state to become `SAFE_MODE`. |
| **Python Analysis** | Recorded telemetry files such as CSV | Processes historical telemetry to visualize spacecraft behaviour, identify trends, and detect anomalies. | Reads battery telemetry and produces a graph showing battery dropping from **95% → 12%**. |
| **CMake** | Source files and build configuration | Compiles and links the different C++ components into the spacecraft simulator executable and test programs. | Builds `spacecraft_simulator.exe` from the Power, Thermal, Fault, State Machine, and other modules. |
| **Documentation** | Requirements, interfaces, state definitions, architecture decisions | Defines how the system is supposed to work and provides technical information for development and maintenance. | Documents that `LOW_BATTERY` must cause `NOMINAL → SAFE_MODE`. |

---

## Example: Normal Operation

The architecture works roughly like this during normal operation:

```text
SENSORS
   │
   │ Temperature = 42°C
   │ Battery = 78%
   │ Altitude = 400 km
   │ Velocity = 7.67 km/s
   ▼
FLIGHT COMPUTER
   │
   │ Processes measurements
   ▼
FAULT MANAGER
   │
   │ No abnormal conditions
   ▼
STATE MACHINE
   │
   │ Remains in NOMINAL
   ▼
TELEMETRY
   │
   │ Records spacecraft status
   ▼
CSV TELEMETRY
   │
   ▼
PYTHON ANALYSIS
   │
   └──► Graphs / statistics / anomaly analysis
```

---

## Example: Low Battery Fault

This example demonstrates **fault handling** within the architecture:

```text
POWER SYSTEM
     │
     │ Battery = 11%
     ▼
FLIGHT COMPUTER
     │
     │ Receives battery status
     ▼
FAULT MANAGER
     │
     │ 11% < 15% threshold
     │
     └──► LOW_BATTERY
              │
              ▼
       STATE MACHINE
              │
              │ NOMINAL → SAFE_MODE
              ▼
         SAFE MODE
              │
              ├── Disable non-essential systems
              ├── Reduce power consumption
              └── Maintain communications
```

### Example Data

```text
Battery:          11%
Temperature:      43°C
Altitude:         400 km
Velocity:         7.67 km/s
Communications:   CONNECTED
CPU Load:         42%

Fault Detected:   LOW_BATTERY
Previous State:   NOMINAL
New State:        SAFE_MODE
```

---

## Example: Sensor Failure

```text
TEMPERATURE SENSOR
        │
        │ Invalid reading
        ▼
FAULT MANAGER
        │
        │ TEMP_SENSOR_FAILURE
        ▼
STATE MACHINE
        │
        │ NOMINAL → SAFE_MODE
        ▼
SAFE MODE
```

---

## Summary

The Spacecraft Simulator architecture is built around a clear data flow:

1. **Sensors** generate measurements.
2. **Subsystems** (Power, Thermal, Communications) process and model the data.
3. The **Flight Computer** makes decisions based on the processed data.
4. The **Fault Manager** identifies problems and converts them into fault events.
5. The **State Machine** determines the spacecraft's operational state based on faults and commands.
6. **Telemetry** records everything for monitoring and analysis.
7. **Python Analysis** processes historical telemetry to visualize trends and detect anomalies.
8. **Tests**, **CMake**, and **Documentation** support development, verification, and maintenance.

This design reflects a realistic **flight-software architecture** where components are interconnected through well-defined data flows, rather than being isolated C++ classes.