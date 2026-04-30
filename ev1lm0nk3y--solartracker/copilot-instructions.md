## solartracker

> This project is a sun tracker apparatus designed to keep portable solar panels optimally aligned with the sun. The system uses a combination of mechanical and electronic components to achieve this.

# GEMINI.md

## Project Overview

This project is a sun tracker apparatus designed to keep portable solar panels optimally aligned with the sun. The system uses a combination of mechanical and electronic components to achieve this.

The mechanical assembly consists of a "lazy susan" base for rotation, driven by a screw drive motor, and a 12VDC linear actuator to control the pitch (vertical angle) of the solar panels.

The electronic control system is based on an Arduino Uno R4 WiFi controller. It uses an Arduino Motor Shield Rev3 to drive both the rotation and pitch motors. Four light detectors provide the necessary input for the sun's position. A 20x4 I2C LCD displays diagnostic information, and a GY-271 (QMC5883L) compass provides heading data. An MPU6050 sensor provides pitch and tilt safety data.

## Building and Running

This is an Arduino-based project. The standard workflow for building and running the code is as follows:

1.  **Install the Arduino IDE:** Download and install the Arduino IDE from the official website (https://www.arduino.cc/en/software).
2.  **Install Libraries:** Install the required libraries using the Arduino IDE's Library Manager (Sketch > Include Library > Manage Libraries...).
    *   `hd44780`: Search for and install the "hd44780" library by Bill Perry (for auto-configured I2C LCD).
    *   `Adafruit MPU6050`: Install the library by Adafruit.
    *   `Adafruit Unified Sensor`: Required dependency for MPU6050.
    *   `QMC5883L`: This library may not be in the default library manager. You can install it by downloading the ZIP file from a source like GitHub (e.g., from user `mprograms`) and then in the IDE, go to Sketch > Include Library > Add .ZIP Library.
3.  **Calibrate Sensors (First Time Only):**
    *   Open `calibrate.ino` in the Arduino IDE.
    *   Upload it to the board.
    *   Open the Serial Monitor (115200 baud).
    *   Follow the on-screen instructions to calibrate the MPU6050 (keep flat) and Compass (rotate).
    *   Copy the outputted code snippets (offsets).
    *   Paste them into the `startupRoutine()` function in `tracker.ino` where indicated.
4.  **Compile & Upload Tracker:**
    *   Open `tracker.ino` in the Arduino IDE.
    *   Click "Verify" to check for errors.
    *   Click "Upload" to flash the code.

## Development Conventions

*   **Code Style:** Follow the standard Arduino C++ coding conventions.
*   **File Structure:** The main sketch file is `tracker.ino`. Utility sketches like `calibrate.ino` are provided for setup.
*   **Libraries:**
    *   `Wire.h` (I2C)
    *   `hd44780.h` & `hd44780ioClass/hd44780_I2Cexp.h` (LCD)
    *   `Adafruit_MPU6050.h` & `Adafruit_Sensor.h` (MPU6050)
    *   `QMC5883L.h` (Compass)
    *   `WiFiS3.h` (WiFi)

## Hardware Details

*   **Manual Mode Switch:** Pin 7 (HIGH = Manual Mode)
*   **Safety Features:**
    *   **Tilt Detection:** Stops motors if tilt > 45 degrees.
    *   **Sensor Monitoring:** Enters manual-only mode if critical sensors (MPU) fail.

---
> Source: [ev1lm0nk3y/solartracker](https://github.com/ev1lm0nk3y/solartracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
