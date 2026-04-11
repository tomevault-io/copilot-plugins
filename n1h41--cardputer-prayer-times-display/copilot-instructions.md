## cardputer-prayer-times-display

> - Upload/flash: `pio run --target upload`

# AGENTS.md

## Build, Lint, and Test Commands
- Build: `pio run`
- Upload/flash: `pio run --target upload`
- Run all tests: `pio test`
- Run a single test: `pio test -f <test_file>`
- Linting: No config found; use `clang-tidy` or `cppcheck` if needed.

## Code Style Guidelines
- Use `#include <...>` for system headers, `#include "..."` for local/project headers.
- Prefer `const` for immutable variables.
- Indent with 2 spaces.
- Use camelCase for variables and functions, PascalCase for classes/types.
- Place error handling in conditional branches; print errors to display or serial.
- Explicitly initialize variables; avoid magic numbers.
- Group related includes together at the top of files.
- Library code: place in `lib/your_library_name/src/`.
- Project code: place in `src/`.
- Tests: place in `test/` and follow PlatformIO unit test conventions.

## PlatformIO Environment & Dependencies
- **Board:** m5stack-stamps3 (ESP32-S3 Cardputer)
- **Platform:** espressif32
- **Framework:** arduino
- **Libraries/Plugins:**
  - M5Cardputer (`m5stack/M5Cardputer@^1.0.3`)
- **platformio.ini:** See file for additional build/upload options.

## Miscellaneous
- No Cursor or Copilot rules present.
- .clangd: custom compile flags removed for compatibility.

For more details, see PlatformIO docs: https://docs.platformio.org/

---

## 🚀 Project Goal: ESP32-S3 Cardputer — Islamic Prayer Times Display

**Objective:**  
Build an embedded application for the ESP32-S3 Cardputer that connects to WiFi, determines location via IP, fetches daily Islamic prayer times from an online API, displays the next prayer and countdown on the device, and provides robust error handling and user feedback.  
This checklist outlines all required steps and features for successful implementation, testing, and documentation.

---

## 📝 Implementation Checklist (Current Status)

- [x] 1. Hardware Setup
    - [x] Confirm ESP32-S3 cardputer is working *(code uses M5Cardputer.h)*
    - [x] Connect display and test basic output *(display prints WiFi and time)*

- [x] 2. PlatformIO Project Initialization
    - [x] Create project directory *(project exists)*
    - [x] Initialize PlatformIO project for ESP32-S3 *(platformio.ini present)*
    - [x] Select correct board in platformio.ini *(board = m5stack-stamps3)*
    - [ ] Verify serial monitor works *(not explicitly shown in code)*

- [x] 3. WiFi Connectivity
    - [x] Add WiFi connection code *(WiFi.begin, status check)*
    - [x] Store/test WiFi credentials *(ssid/password in code)*
    - [x] Handle connection errors *(status checked, error printed)*

- [x] 4. Time Synchronization
    - [x] Integrate NTP client for accurate time *(configTime, getLocalTime used)*
    - [x] Confirm device time matches local time zone *(time printed to display)*

- [ ] 5. IP Geolocation
    - [ ] Add HTTPClient and ArduinoJson libraries
    - [ ] Fetch location (lat/lon/city) via IP geolocation API
    - [ ] Parse and store location data
    - [ ] Handle API errors and rate limits

- [ ] 6. Prayer Times API Integration
    - [ ] Choose and test prayer times API (e.g., AlAdhan)
    - [ ] Fetch today’s prayer times using lat/lon
    - [ ] Parse prayer times from API response
    - [ ] Store prayer times for comparison

- [ ] 7. Next Prayer Calculation
    - [ ] Compare current time to prayer times
    - [ ] Determine next upcoming prayer (name & time)
    - [ ] Calculate countdown to next prayer

- [x] 8. Display Integration
    - [x] Initialize display library for cardputer *(M5Cardputer.Display used)*
    - [ ] Design layout for prayer times and next prayer
    - [ ] Display next prayer name, time, and countdown
    - [ ] Optionally display all prayer times for today

- [ ] 9. Periodic Updates & Refresh Logic
    - [ ] Refresh location and prayer times at intervals (e.g., hourly, midnight)
    - [x] Update display automatically *(time updates every second)*
    - [ ] Optionally add manual refresh (button)

- [x] 10. Error Handling & User Feedback
    - [x] Display connection/API errors on screen *(WiFi and NTP errors printed)*
    - [ ] Handle WiFi disconnects gracefully
    - [ ] Fallback to last known prayer times if API fails

- [ ] 11. Optional Features
    - [ ] Show Hijri date
    - [ ] Support different calculation methods
    - [ ] User settings (method, location override)
    - [ ] Display all prayer times for today

- [ ] 12. Testing & Validation
    - [ ] Test with different locations (simulate IP)
    - [ ] Test display output for all prayer times
    - [ ] Test error cases (no WiFi, API down)
    - [ ] Validate time zone and prayer time accuracy

- [ ] 13. Documentation
    - [ ] Document setup steps and dependencies
    - [ ] Explain WiFi and API configuration
    - [ ] Provide user guide for settings and usage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/n1h41)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/n1h41)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
