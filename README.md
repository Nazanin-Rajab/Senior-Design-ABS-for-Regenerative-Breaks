# ABS for Regenerative Brakes on Electric Skateboard

This project implements an experimental anti-lock braking (ABS) system for an electric skateboard with regenerative braking. It uses an ESP32 with an IMU to sense board dynamics and a Bluetooth Low Energy (BLE) link to a laptop Python client for live data streaming, visualization, and control.

Developed as part of a senior design project at the Georgia Institute of Technology.

***

## Project Overview

The goal of this project is to detect the onset of wheel slip during braking and modulate braking current to reduce stopping distance while maintaining stability.

**Key elements:**

* ESP32 microcontroller mounted on the skateboard
* ICM-20948 IMU to measure acceleration
* BLE link between ESP32 and laptop
* Python client using `bleak` to read IMU data and send commands
* Integration with a VESC motor controller (UART + CAN) for wheel control and ABS logic

A detailed engineering notebook with day-by-day progress, findings, and code evolution is included in this repository (see `docs/` or `notebook.md`, depending on how you organize it).

***

## System Architecture

**High-level architecture:**

**ESP32**
* Acts as a BLE server advertising as `SkateABS`
* Reads IMU data via I2C from ICM-20948
* Calibrates and filters acceleration data
* Streams filtered acceleration to the laptop over BLE
* Receives text commands from the laptop over BLE

**Laptop (Python BLE Client)**
* Discovers and connects to `SkateABS`
* Subscribes to BLE notifications for IMU data
* Logs incoming data with timestamps
* Provides an interactive terminal for sending commands back to ESP32

**VESC (Motor Controller) – Integrated in later stages**
* Communicates with ESP32 over UART (and CAN forwarding)
* Provides wheel speed, current, and voltage
* Executes commanded braking currents for ABS testing

***

## Features

* BLE communication between ESP32 and laptop with custom BLE service and characteristic
* Read, write, and notify support
* Real-time IMU data acquisition using ICM-20948
* Calibration routine to compute static gravity offsets
* High-pass filtering + moving average smoothing on acceleration
* BLE notifications with formatted dynamic acceleration data

***

## Python Client Setup

1. Ensure your laptop has BLE hardware enabled.

2. Install Python dependencies (if you haven't already):
```bash
pip install bleak
```

3. Run the client:
```bash
python ble_client.py
```

The script will:
* Scan and list nearby BLE devices
* Connect to the device named `SkateABS`
* Start printing notifications with timestamps as data arrives
* Allow you to type messages in the terminal to send back to the ESP32 over BLE

***

## ABS Logic (Summary)

From the integrated firmware documented in the project notebook.

### Slip Detection

The slip detection logic operates on wheel speed (MPH) reported by the VESC:

* Monitor wheel speed (MPH) at each sample
* Compute the drop in speed between consecutive samples
* If the speed drop exceeds a threshold (e.g., > 10 MPH) while braking:
  * Classify the event as slip
  * Trigger the ABS ramp behavior

### Ramp Algorithm

The ABS ramp algorithm modulates braking current in three phases:

**Phase 1 – Drive Phase**
* Apply a constant motor current: `rampMotorCurrent`
* Maintain this for a fixed duration: `rampDuration`

**Phase 2 – Full-Brake and Slip Detection Phase**
* Apply full braking current: `rampEndBrakeCurrent`
* Intentionally allow the wheel to lock to provoke slip
* Continuously monitor wheel speed for a >10 MPH drop
* When slip is detected, transition to the ramp phase

**Phase 3 – 4-Step Ramp Phase**
* Reduce braking current to a lower fallback value: `rampStartBrakeCurrent`
* Gradually ramp the brake current from `rampStartBrakeCurrent` back up to `rampEndBrakeCurrent` in four steps over `rampUpTime`

**Goal:** Limit wheel lock, maintain traction, and reduce stopping distance while still leveraging regenerative braking.

For exact implementation details and parameter values, see the project notebook (`notebook.md` or `docs/project_log.md`).

***

## Repository Structure

Example layout:
├── esp32.ino              ESP32 firmware
├── ble_client.py          Python BLE client

***

## Team

* **Nazanin Rajabi** (Computer Engineering) – BLE, ESP32, IMU integration, ABS control logic, Python client
* **Asil Poyraz Yongaci** (Electrical Engineering)
* **Connor Ciesielski** (Electrical Engineering)
* **Kyle Kamani** (Electrical Engineering)
* **Shanmathi Selvamurugan** (Computer Engineering)
* **Kenny Dang** (Computer Engineering)
