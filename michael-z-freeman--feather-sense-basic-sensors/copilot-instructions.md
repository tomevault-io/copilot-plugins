## feather-sense-basic-sensors

> This project reads sensor data from the **Adafruit Feather nRF52840 Sense** and prints it to the Serial monitor.

# Feather Sense Basic Sensors

This project reads sensor data from the **Adafruit Feather nRF52840 Sense** and prints it to the Serial monitor.

## Development Setup

The project uses `arduino-cli` for compilation and library management.

### Prerequisites

1.  **Install arduino-cli** (via Homebrew on macOS):
    ```bash
    brew install arduino-cli
    ```

2.  **Initialize Configuration**:
    ```bash
    arduino-cli config init
    ```

3.  **Add Adafruit Board Index**:
    ```bash
    arduino-cli core update-index --additional-urls https://adafruit.github.io/arduino-board-index/package_adafruit_index.json
    ```

4.  **Install Adafruit nRF52 Core**:
    ```bash
    arduino-cli core install adafruit:nrf52
    ```

### Required Libraries

Install the sensor libraries using `arduino-cli`:

```bash
arduino-cli lib install \
  "Adafruit APDS9960 Library" \
  "Adafruit BMP280 Library" \
  "Adafruit LIS3MDL" \
  "Adafruit LSM6DS" \
  "Adafruit SHT31 Library" \
  "Adafruit Unified Sensor" \
  "Adafruit BusIO"
```

## Compilation

To compile the sketch for the Adafruit Feather nRF52840 Sense:

```bash
arduino-cli compile --fqbn adafruit:nrf52:feather52840sense Feather_Sense_basic_sensors.ino
```

## Upload

To upload to the board (replace `/dev/cu.usbmodemXXXX` with your actual port):

```bash
arduino-cli upload -p /dev/cu.usbmodemXXXX --fqbn adafruit:nrf52:feather52840sense Feather_Sense_basic_sensors.ino
```

## Sensors Covered
- **APDS9960**: Proximity, Light, Color, Gesture
- **BMP280**: Temperature, Barometric Pressure
- **LIS3MDL**: Magnetometer
- **LSM6DS33 / LSM6DS3TR-C**: Accelerometer, Gyroscope
- **SHT31**: Humidity
- **PDM**: Microphone

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Michael-Z-Freeman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
