# ABS for Regenerative Brakes on Electric Skateboard

This project implements an experimental anti-lock braking (ABS) system for an electric skateboard with regenerative braking.  
It uses an ESP32 with an IMU to sense board dynamics and a Bluetooth Low Energy (BLE) link to a laptop Python client for live data streaming, visualization, and control.

Developed as part of a senior design project at the Georgia Institute of Technology.

---

## Project Overview

The goal of this project is to detect the onset of wheel slip during braking and modulate braking current to reduce stopping distance while maintaining stability.

Key elements:

- ESP32 microcontroller mounted on the skateboard
- ICM-20948 IMU to measure acceleration
- BLE link between ESP32 and laptop
- Python client using `bleak` to read IMU data and send commands
- Integration with a VESC motor controller (UART + CAN) for wheel control and ABS logic

A detailed engineering notebook with day-by-day progress, findings, and code evolution is included in this repository (see `docs/` or `notebook.md`, depending on how you organize it).

---

## System Architecture

High-level architecture:

- **ESP32**
  - Acts as a BLE server advertising as `SkateABS`
  - Reads IMU data via I2C from ICM-20948
  - Calibrates and filters acceleration data
  - Streams filtered acceleration to the laptop over BLE
  - Receives text commands from the laptop over BLE

- **Laptop (Python BLE Client)**
  - Discovers and connects to `SkateABS`
  - Subscribes to BLE notifications for IMU data
  - Logs incoming data with timestamps
  - Provides an interactive terminal for sending commands back to ESP32

- **VESC (Motor Controller) – Integrated in later stages**
  - Communicates with ESP32 over UART (and CAN forwarding)
  - Provides wheel speed, current, and voltage
  - Executes commanded braking currents for ABS testing

---

## Features

- BLE communication between ESP32 and laptop
  - Custom BLE service and characteristic
  - Read, write, and notify support
- Real-time IMU data acquisition using ICM-20948
- Calibration routine to compute static gravity offsets
- High-pass filtering + moving average smoothing on acceleration
- BLE notifications with formatted dynamic acceleratio
