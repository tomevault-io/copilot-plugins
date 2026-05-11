## project-starbeam

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Quick Reference Commands

```bash
# Compile and upload V2 firmware (ALWAYS use huge_app partition!)
arduino-cli compile --upload --fqbn esp32:esp32:esp32:PartitionScheme=huge_app -p /dev/cu.SLAB_USBtoUART starbeam_v2/

# Test compilation without hardware
./test_build.sh

# Monitor serial output
screen /dev/cu.SLAB_USBtoUART 115200

# Port names: macOS=/dev/cu.*, Linux=/dev/ttyUSB0, Windows=COM3
```

## Project Overview

Project Starbeam V2 is a multi-band RF signal intelligence platform built around the ESP32-WROOM-32D microcontroller with multiple RF modules (NRF24L01 and CC1101) for signal analysis, generation, and manipulation across 2.4 GHz and 433 MHz bands. It includes optional HackRF One integration for extended frequency coverage up to 6 GHz.

**CRITICAL LEGAL NOTICE**: This is a security research and educational platform. RF transmission, jamming operations, and WiFi security testing features are strictly regulated. Only operate on permitted frequencies with proper authorization. This tool is intended for authorized security testing, educational contexts, and research environments.

## Important: Repository Structure

This repository contains **two firmware versions**:

- **starbeam_v1/** - Original monolithic firmware (~1000+ line single .ino file)
- **starbeam_v2/** - Modular refactored firmware (current development focus)

Unless explicitly working on V1, **all development should target starbeam_v2/**.

## Development Environment Setup

### Arduino IDE Configuration

1. **Install Arduino IDE 2.x** from https://www.arduino.cc/en/software/
2. **Add ESP32 Board Support**:
   - File → Preferences → Additional Board Manager URLs
   - Add: `https://dl.espressif.com/dl/package_esp32_index.json`
   - Tools → Board → Boards Manager → Install "esp32" by Espressif Systems

3. **Required Libraries** (Install via Arduino Library Manager):
   - `Adafruit GFX Library`
   - `Adafruit SSD1306`
   - `U8g2_for_Adafruit_GFX`
   - `RF24` by TMRh20
   - `ELECHOUSE_CC1101_SRC_DRV` by LSatan
   - `ESP32 BLE Arduino` (included with ESP32 board support)

4. **Manual Library Installation**:
   - The `SmartRC-CC1101-Driver-Lib2/` directory is already in the repository
   - Dual CC1101 radio support via `ELECHOUSE_CC1101_SRC_DRV2.h`

### Board Configuration

- **Board**: ESP32 Dev Module
- **Upload Speed**: 921600
- **CPU Frequency**: 240MHz
- **Flash Frequency**: 80MHz
- **Flash Mode**: QIO
- **Flash Size**: 4MB (32Mb)
- **Partition Scheme**: **Huge APP (3MB No OTA/1MB SPIFFS)** - REQUIRED for security features
- **Core Debug Level**: None (or "Verbose" for debugging)

**IMPORTANT**: The `huge_app` partition scheme is **required** for V2 firmware. It allocates 3MB for program storage instead of the default 1.2MB, which is necessary for WiFi attack features and web server functionality.

### Building and Uploading

#### Arduino CLI (Recommended)

```bash
# Install Arduino CLI (if not already installed)
# macOS: brew install arduino-cli
# Linux: curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | sh
# Windows: Download from https://arduino.github.io/arduino-cli/

# Install ESP32 platform (first time only)
arduino-cli core update-index
arduino-cli core install esp32:esp32

# Install required libraries (first time only)
arduino-cli lib install "Adafruit GFX Library"
arduino-cli lib install "Adafruit SSD1306"
arduino-cli lib install "U8g2_for_Adafruit_GFX"
arduino-cli lib install "RF24"
arduino-cli lib install "ELECHOUSE_CC1101_SRC_DRV"

# Compile only (no upload)
arduino-cli compile --fqbn esp32:esp32:esp32:PartitionScheme=huge_app starbeam_v2/

# Compile and upload in one command
arduino-cli compile --upload --fqbn esp32:esp32:esp32:PartitionScheme=huge_app -p /dev/cu.SLAB_USBtoUART starbeam_v2/

# Or separate compile and upload
arduino-cli compile --fqbn esp32:esp32:esp32:PartitionScheme=huge_app starbeam_v2/
arduino-cli upload --fqbn esp32:esp32:esp32:PartitionScheme=huge_app -p /dev/cu.SLAB_USBtoUART starbeam_v2/

# Port examples:
# macOS:   /dev/cu.SLAB_USBtoUART or /dev/cu.usbserial-*
# Linux:   /dev/ttyUSB0 or /dev/ttyACM0
# Windows: COM3 or COM4 (check Device Manager)
```

#### Arduino IDE

```bash
# Open: starbeam_v2/starbeam_v2.ino
# Tools → Board → ESP32 Arduino → ESP32 Dev Module
# Tools → Partition Scheme → Huge APP (3MB No OTA/1MB SPIFFS)  ← REQUIRED
# Tools → Port → Select your ESP32 port
# Sketch → Upload (Ctrl+U / Cmd+U)
# Hold BOOT button on ESP32 during upload if auto-reset fails
```

#### Automated Build Testing (No Hardware)

```bash
# Run from project root directory
./test_build.sh

# This script will:
# - Check Arduino CLI installation
# - Verify ESP32 platform is installed
# - Auto-install missing libraries
# - Compile firmware with huge_app partition
# - Report memory usage (should be ~1.81 MB / 3 MB)
```

### Serial Monitor

```bash
# Arduino IDE: Tools → Serial Monitor, set to 115200 baud

# External serial monitor:
screen /dev/cu.usbserial-* 115200
# or
minicom -D /dev/cu.usbserial-* -b 115200
```

## Code Architecture (V2)

### Modular Component Structure

**V2 uses a clean modular architecture** with functionality split across focused files:

**Core Configuration** (`starbeam_v2/`):
- `starbeam_v2.ino` - Main firmware, state machine, menu system
- `config.h` - Hardware pin definitions, buffer sizes, timing constants, FreeRTOS config
- `types.h` - Enums (AppState, MenuItem), structs (SignalInfo, RadioConfig)

**Functional Modules** (`starbeam_v2/src/`):
- `display.h/cpp` - SSD1306 OLED interface, U8g2 fonts, menu rendering
- `input.h/cpp` - Button handling with debouncing, long-press detection
- `nrf24.h/cpp` - NRF24L01 radio management (5 radios across 2 SPI buses)
- `cc1101.h/cpp` - CC1101 radio control (2 radios, frequency scanning, RSSI)
- `analyzer.h/cpp` - Standalone NRF24 spectrum analyzer with signal graphing
- `recording.h/cpp` - Signal capture/replay, EEPROM persistence
- `wifi_scanner.h/cpp` - WiFi network enumeration using ESP32 internal WiFi
- `ble_scanner.h/cpp` - BLE device discovery using ESP32 internal Bluetooth
- `webserver.h/cpp` - Web interface, captive portal, REST APIs
- `wifi_attack.h/cpp` - WiFi security testing (deauth, beacon flood, probe flood, PMKID)

### Key Architectural Patterns

1. **Modular Design**: Each radio/feature in separate .h/.cpp files (V1 was monolithic)
2. **Dual SPI Bus Architecture**:
   - VSPI: NRF24 radios 1-3 (CE=27/26/25, CS=15/33/5)
   - HSPI: NRF24 radios 4-5 (CE=4/32, CS=2/17), CC1101 radios 1-2 (SS=2/32)
3. **State Machine**: Menu-driven with ~30+ operational states (AppState enum in types.h)
4. **Non-blocking Operations**: Uses `nonBlockingDelay()` and `yield()` for responsiveness
5. **Static Buffer Management**: Fixed-size buffers prevent heap fragmentation
   - CCBUFFERSIZE=64, RECORDINGBUFFERSIZE=4096, EEPROM_SIZE=512
6. **IEEE 802.11 Raw Frame Injection**: Direct `esp_wifi_80211_tx()` for security testing
7. **Promiscuous Mode Capture**: IRAM_ATTR callbacks for packet sniffing

### Critical Hardware Abstraction

**Radio Module Configuration** (defined in `config.h`):
- **NRF24 Radios 1-3**: VSPI bus, 2.4 GHz (Bluetooth/WiFi jamming, scanning)
- **NRF24 Radios 4-5**: HSPI bus, configurable (can be replaced with CC1101)
- **CC1101 Radios 1-2**: HSPI bus, 300-928 MHz (ISM band jamming, scanning)
- **Display**: SSD1306 OLED, 128x64, I2C address 0x3C
- **Controls**: GPIO 39 (Up), 34 (Down), 36 (Select)

**CC1101 Pin Mappings**:
- CC1101 #1: SCK=14, MISO=12, MOSI=13, SS=2, GDO0=4, GDO2=16
- CC1101 #2: SCK=14, MISO=12, MOSI=13, SS=32, GDO0=35, GDO2=17
- Both share SPI bus but use separate chip selects

### WiFi Attack Framework (NEW in V2)

**Core Components**:
- `wifi_attack.h` - Attack type definitions, IEEE 802.11 frame structures
- `wifi_attack.cpp` - Raw frame injection, promiscuous mode sniffing
- `esp_wifi_80211_tx()` - Low-level ESP32 API for frame transmission
- `esp_wifi_set_promiscuous()` - Monitor mode for packet capture

**Frame Types**:
- **Deauth**: Type 0xC0, configurable reason codes (1, 3, 7)
- **Beacon**: Type 0x80, complete IE construction (SSID, rates, DS param)
- **Probe**: Type 0x40, random MAC generation with proper bit flags
- **EAPOL**: RSN IE parsing for PMKID extraction

**Attack Modes**:
1. **Targeted Deauth**: Sniffs specific AP, disconnects clients (16 frames/client)
2. **Broadcast Deauth**: Channel hopping (1-13), affects all networks
3. **Beacon Flood**: 20 fake APs, >100 beacons/sec
4. **Probe Flood**: Random MAC, >500 probes/sec
5. **PMKID Capture**: WPA2/WPA3 handshake sniffing, hashcat output format

## Development Workflow

### Testing Without Hardware

Use the automated build test to verify compilation without physical hardware:

```bash
./test_build.sh
```

This will:
- Check for Arduino CLI installation
- Install missing libraries
- Compile the project
- Report memory usage (flash/RAM)
- Show compilation errors

**Expected Output**:
```
======================================
✓ COMPILATION SUCCESSFUL
======================================
Binary size: 1.81 MB (58%)
Sketch uses ~892KB flash, ~45KB RAM
Build test PASSED ✓
```

### Adding New Features

**Adding New Menu Items**:
1. Add state to `AppState` enum in `types.h`
2. Add menu item to `MenuItem` enum in `types.h`
3. Implement state handler in main `.ino` file
4. Add display logic for the operation
5. Test with serial monitor for debug output

**Adding New Radio Operations**:
1. Create new module in `src/` (e.g., `new_feature.h/cpp`)
2. Include in main `.ino` file
3. Add initialization in `setup()`
4. Call from state machine in `loop()`

**Adding New Web Interface Features**:
1. Add handler function in `webserver.cpp`
2. Register route in `StarbeamWebServer::start()`
3. Add HTML generation in helper functions
4. Create JSON API endpoint for AJAX updates

### Frequency Configuration

CC1101 frequency presets (in menu system):
- 433.92 MHz (STATE_SET_43392)
- 434.00 MHz (STATE_SET_43400)
- 433.90 MHz (STATE_SET_43390)
- 433.87 MHz (STATE_SET_43387)
- 388.00 MHz, 390.00 MHz, 400.00 MHz, 434.50 MHz, 434.40 MHz, 434.30 MHz

Modulation modes: 2-FSK, GFSK, ASK/OOK, 4-FSK, MSK
Frequency range: 300-348 MHz, 387-464 MHz, 779-928 MHz

### Debug and Monitoring

Serial output provides real-time diagnostics (115200 baud):
- Boot sequence and module initialization
- WiFi/BLE scan results with MAC addresses
- Attack statistics (frames sent, clients disconnected)
- PMKID data in hashcat format: `PMKID*BSSID*STATION*ESSID`
- RSSI values (CC1101 uses 2's complement dBm, NRF24 uses RPD register)

## Security Testing Features

**CRITICAL**: All security testing features require explicit authorization. See README.md for legal compliance requirements.

**Deauthentication Attacks**:
- Targeted: Sniffs target AP, disconnects detected clients
- Broadcast: Channel hopping, affects all nearby networks
- Extra confirmation required for broadcast mode

**Beacon Flooding**:
- Generates up to 20 fake APs with random BSSIDs
- Tests AP isolation and scanner resilience

**Probe Request Flooding**:
- Simulates multiple client devices
- Random MAC address generation (locally administered)

**PMKID Capture**:
- Extract PMKID from WPA2/WPA3 EAPOL handshakes
- Hashcat-compatible output for educational analysis
- 2-minute timeout with WPA2/3 filtering

## Web Interface

Access via built-in access point:
1. Select "Web Server" from menu
2. Connect to `StarbeamAP` WiFi (password: `starbeam2024`)
3. Navigate to `http://192.168.4.1`

**Endpoints**:
- `/` - Home dashboard
- `/wifi` - WiFi scanner page
- `/ble` - BLE scanner page
- `/security` - Security testing dashboard
- `/api/wifi` - WiFi scan results JSON
- `/api/ble` - BLE scan results JSON
- `/api/attack/status` - Attack status JSON

## Common Issues

### Upload Failures
- Hold BOOT button during upload if auto-reset fails
- Verify correct port selection
- Reduce upload speed to 460800 if 921600 fails

### Compilation Errors
- Run `./test_build.sh` to auto-install missing libraries
- Verify ESP32 board support is installed
- Check partition scheme (use `huge_app` for full feature set)

### Radio Module Issues
- Verify correct SPI bus assignment (VSPI vs HSPI in config.h)
- Check that CC1101 library variant 2 is available in `SmartRC-CC1101-Driver-Lib2/`
- Ensure antennas are connected before transmission (prevents PA damage)

### Display Not Working
- Verify I2C address is 0x3C
- Check SDA/SCL connections (default ESP32 I2C pins 21/22)
- Some displays require SSD1306_SWITCHCAPVCC vs external VCC

### WiFi Attack Issues
- Ensure partition scheme is `huge_app` (security features use more flash)
- Verify WiFi is initialized before attack operations
- Check serial monitor for error messages during frame injection

### Memory Issues
- Monitor free heap: `ESP.getFreeHeap()` (should be >200 KB)
- Check task stack sizes in config.h if adding FreeRTOS tasks
- Static buffers prevent fragmentation but use fixed RAM

## Hardware Design Files

Located in `hardware/starbeam_V1_design/`:
- **Gerber Files**: `Gerber_starbeam_V1_PCB.zip` (for PCB manufacture)
- **Schematic**: `Schematic_starbeam_V1.pdf`
- **BOM**: `BOM_starbeam_V1.csv`
- **Pick-and-Place**: `PickAndPlace_PCB_starbeam_V1.csv`
- **Stackup**: 4-layer PCB with JLCPCB default stackup

Note: V2 firmware is 100% backward compatible with V1 hardware.

## Performance Metrics

**Firmware Statistics** (V2):
- Flash Usage: 1.81 MB (58% of 3MB)
- RAM Usage: ~70 KB (21% of 320KB)
- Free Heap: >200 KB
- Boot Time: ~2.5s

**Attack Performance**:
- Beacon Flood: >100 beacons/second
- Probe Flood: >500 probes/second
- Deauth (Targeted): Real-time client detection and elimination
- Deauth (Broadcast): Channel hopping with full coverage
- PMKID Parse: <100ms per EAPOL frame

## Resources

- Arduino CLI Installation: https://arduino.github.io/arduino-cli/
- ESP32 Documentation: https://docs.espressif.com/projects/esp-idf/en/latest/esp32/
- NRF24L01 Library: https://github.com/nRF24/RF24
- CC1101 Library: https://github.com/LSatan/SmartRC-CC1101-Driver-Lib
- HackRF Documentation: https://hackrf.readthedocs.io/

## License

Apache License 2.0 - See LICENSE file for details.

---
> Source: [dkyazzentwatwa/project-starbeam](https://github.com/dkyazzentwatwa/project-starbeam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
