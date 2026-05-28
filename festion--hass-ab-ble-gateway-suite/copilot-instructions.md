## hass-ab-ble-gateway-suite

> Claude should help users with the following tasks related to this Home Assistant integration suite for AprilBrother BLE Gateways and BLE device discovery:

# AprilBrother BLE Gateway Suite for Home Assistant

## How Claude Should Assist

Claude should help users with the following tasks related to this Home Assistant integration suite for AprilBrother BLE Gateways and BLE device discovery:

### Home Assistant Integration Component
- Modifications to Python code in the custom component
- Configuration flow adjustments
- MQTT integration enhancements
- Scanner functionality improvements
- Service definition updates

### BLE Discovery Add-on
- Modifications to Python code in `ble_discovery.py`
- Adjustments to YAML configuration files
- Docker-related configuration and build issues
- Add-on UI customization

## Key Commands to Run

### Custom Component
- Format: `black custom_components/`
- Lint: `flake8 custom_components/`
- Import sorting: `isort custom_components/`
- Test: `pytest tests/`

### BLE Discovery Add-on
- Lint and check Python code: `flake8 addon/ble_discovery.py`
- Run the add-on locally: `./addon/run.sh`
- Build Docker image: `docker build -t ble-discovery-addon ./addon`

## Important Files

### Custom Component
- `custom_components/ab_ble_gateway/`: Main component directory
- `custom_components/ab_ble_gateway/__init__.py`: Component initialization
- `custom_components/ab_ble_gateway/config_flow.py`: Configuration UI flow
- `custom_components/ab_ble_gateway/scanner.py`: BLE gateway scanner
- `custom_components/ab_ble_gateway/const.py`: Component constants

### BLE Discovery Add-on
- `addon/ble_discovery.py`: Main Python code for BLE device discovery
- `addon/ble_input_text.yaml`, `addon/ble_scripts.yaml`, `addon/btle_dashboard.yaml`: YAML configuration files
- `addon/config.json`: Add-on configuration
- `addon/Dockerfile`: Container build configuration
- `addon/run.sh`: Script to run the add-on

### Dashboard Files
- `btle_combined_dashboard.yaml`: Combined dashboard for BLE device discovery and gateway management
- `btle_dashboard.yaml`: Dashboard for BLE device discovery
- `btle_gateway_management.yaml`: Dashboard for BLE gateway management

## Dashboard Installation

To manually install the dashboard:

1. Go to Home Assistant UI
2. Navigate to Settings > Dashboards
3. Click "Add Dashboard" then select "From YAML"
4. Copy the contents from one of the dashboard YAML files
5. Save the dashboard

### Dashboard Options

1. **Basic Dashboard** (`basic_dashboard.yaml`):
   - Simple status display with gateway attributes
   - MQTT reconnect button
   - Detected devices counter
   - **Recommended for troubleshooting**

2. **Minimal Dashboard** (`minimal_dashboard.yaml`):
   - Ultra-simple view with only raw data
   - MQTT reconnect button
   - Device addition form
   - **Best for initial testing**

3. **Combined Dashboard** (`btle_combined_dashboard.yaml`):
   - Full-featured dashboard with all functionality
   - Multi-view structure with tabs
   - Device discovery and management
   - Gateway status monitoring
   - **Use once basic functionality is confirmed**

### Troubleshooting Dashboard Issues

1. **Entity Not Available**:
   - Make sure integration is correctly installed
   - Verify gateway is sending data (check MQTT Explorer)
   - Try the MQTT reconnect button
   - Restart Home Assistant if needed

2. **Missing Input Helpers**:
   - Create them manually if not automatically created:
     - `input_text.new_ble_device_name`
     - `input_text.new_ble_device_mac`
     - `input_text.new_ble_device_category`

3. **Dashboard Not Visible**:
   - Ensure "Show in sidebar" is enabled
   - Check browser cache or try different browser

4. **Template Syntax Errors**:
   - If dashboard shows errors like `TemplateSyntaxError: expected token ')', got '['`, 
     ensure multi-line JavaScript selectors in templates are on single lines
   - This commonly occurs in query selectors with brackets in device selection code

5. **Null Data Display**:
   - Verify gateway configuration
   - Check MQTT connection
   - Inspect raw payload format
   
6. **Script Issues**:
   - If you see errors like `TypeError: unsupported operand type(s) for +: 'int' and 'str'` in scripts,
     ensure proper type conversion using filters like `|int(-100)` when working with RSSI values
   - If errors mention `Referenced entities button.bluetooth_scan are missing`, the component 
     includes fallback mechanisms to handle missing button entities

## Key Concepts
- AprilBrother BLE Gateway integration
- BLE device scanning and discovery
- MQTT integration with Home Assistant
- Signal strength (RSSI) threshold settings
- Dashboard UI components
- Device categorization and management

## Style Guidelines
- Follow Home Assistant custom component conventions
- Line length: 88 characters (black default)
- Imports: Use isort with sections (stdlib, third-party, first-party)
- Type hints: Required for function parameters and return values
- Error handling: Use try/except blocks with specific exceptions, log errors
- Naming: snake_case for variables/functions, PascalCase for classes
- String formatting: Use f-strings

## Current Improvements (2024-04-13 - v0.3.22)

### Fixed Issues:
1. **Home Assistant Restart on Gateway Reconnect**
   - Fixed the MQTT reconnect service that was causing Home Assistant to restart
   - Implemented improved error handling for MQTT message processing
   - Added safe wrapper for scanner MQTT handlers
   - Added empty message payload check to prevent processing errors
   - Removed automatic restart from device addition scripts (v1.4.1)
   - Improved lock handling to prevent concurrent reconnects (v0.2.8)
   - Added a static MQTT handler to avoid duplicate subscriptions (v0.2.8)
   - Fixed argument mismatch in advertisement processing (v0.2.9)
   - Added 30-second cooldown between reconnect attempts (v0.3.0)
   - Implemented global reconnect state tracking to prevent concurrent reconnects (v0.3.0)
   - Fixed syntax error in _async_on_advertisement call (v0.3.1)
   - Changed keyword arguments to positional arguments in _async_on_advertisement call (v0.3.2)
   - Added fallback to older API format for _async_on_advertisement (v0.3.3)
   - Implemented comprehensive multi-format support with progressive fallbacks for timestamps (v0.3.4)

2. **"Extracted data is not a dictionary: <class 'int'>" Error**
   - Added better error handling in MQTT message processing
   - MQTT messages with integer payloads now handled gracefully
   - Added additional defensive coding in the message handler
   - Enhanced JSON parsing with verbose logging for troubleshooting (v0.2.8)
   - Added detailed device data format validation and logging (v0.2.8)

3. **JSON Payload Handling**
   - Added support for JSON-formatted payloads from BLE gateways
   - Improved extraction of device data from different payload formats
   - Enhanced metadata extraction from payloads
   - Added device mapping support for friendly device names

4. **Dashboard Missing from Sidebar**
   - Created multiple dashboard options with progressive complexity
   - Added documentation on dashboard installation process
   - Created verification dashboard for entity troubleshooting
   - Added defensive templating to handle missing attributes
   - Implemented simplified dashboard options for easier troubleshooting
   - Fixed template errors in verification dashboard (v0.3.8)
   - Added explicit attribute checks to prevent template rendering issues (v0.3.8)
   - Fixed BLE Discovery add-on errors with timedelta and headers references (v0.3.9, v1.5.3)
   - Completely rebuilt verification dashboard using entities card to avoid Jinja2 template errors (v0.3.10, v1.5.4)
   - Fixed persistent datetime.timedelta and headers reference issues in BLE Discovery add-on (v0.3.11, v1.5.5)
   - Fixed remaining templating error in atomic dashboard by replacing Jinja loop with entities card (v0.3.12, v1.5.6)
   - Comprehensively fixed persistent errors in BLE Discovery add-on by removing redundant timedelta import and ensuring proper headers initialization (v0.3.13, v1.5.7)
   - Added direct datetime module import to fix persistent "datetime.timedelta" errors (v0.3.15, v1.5.8)
   - Added global headers variable in main loop to prevent "headers referenced before assignment" errors (v0.3.15, v1.5.8)
   - Added automatic creation of required dashboard input_text entities (v0.3.15, v1.5.8)
   - Completely rewrote datetime handling with safe wrapper functions (v0.3.16, v1.5.9)
   - Added global variable declarations and nested function to fix persistent headers reference errors (v0.3.16, v1.5.9)
   - Moved scan interval sensor update to a dedicated self-contained function (v0.3.16, v1.5.9)
   - Completely rewrote datetime handling by removing wrapper functions and using direct module calls (v0.3.17, v1.6.0)
   - Removed activity level calculation to avoid datetime issues (v0.3.17, v1.6.0)
   - Removed scan interval sensor creation to eliminate headers reference errors (v0.3.17, v1.6.0)
   - Eliminated global variable declarations in favor of local variables (v0.3.17, v1.6.0)
   - Fixed template variable warnings for undefined mac_address and device_name in scripts (v0.3.18, v1.6.1)
   - Fixed device discovery issues where add-on reported "0 devices" despite valid MQTT payload (v0.3.19, v1.6.2)
   - Enhanced MQTT sensor discovery with comprehensive scanning of all sensor entities (v0.3.20, v1.6.3)

5. **"Error getting MQTT topics" Bug**
   - Fixed error: "argument of type 'bool' is not iterable"
   - Added type checking for hass.data[DOMAIN] in reconnect service
   - Improved error handling in MQTT topic retrieval
   - Added defensive coding against non-dictionary values
   - Fixed global MQTT message handler to properly check data types before iteration (v0.3.14)
   - Added comprehensive type checking for domain data and entry data (v0.3.14)
   - Improved device name lookup with proper type safety checks (v0.3.14)
   - Fixed critical "argument of type 'bool' is not iterable" error in MQTT message handler (v0.3.21)

6. **Device Discovery Issues**
   - Fixed issues where add-on reported finding 0 devices despite receiving valid MQTT payloads (v0.3.19, v1.6.2)
   - Enhanced AprilBrother Gateway payload format detection and processing (v0.3.19, v1.6.2)
   - Improved handling of JSON payload format containing nested device arrays (v0.3.19, v1.6.2)
   - Added proper creation of device entries from Gateway format data (v0.3.19, v1.6.2)
   - Added specific handling for the format: `{"v":1,"mid":12,"time":1744564900,"ip":"192.168.1.82","mac":"E831CDCCCBB0","devices":[[0,"D712ED6A66C6",-85,"0201061AFF4C000215B5B182C7EAB14988AA99B5C1517008D90001C666C5"],[0,"D0E29D3E51BA",-91,"0201061AFF4C000215B5B182C7EAB14988AA99B5C1517008D90001BA51C5"]],"rssi":-43,"metadata":{...}}` (v0.3.19, v1.6.2)
   - Further enhanced discovery by checking all sensor entities (not just those with specific names) (v0.3.20, v1.6.3)
   - Added detailed debugging of all MQTT entities to show device formats (v0.3.20, v1.6.3)
   - Added direct scanning of MQTT sensor states for device data regardless of entity naming (v0.3.20, v1.6.3)
   - Fixed template syntax errors in device selection code (v0.3.23, v1.6.5)
   - Fixed type errors in BLE signal test script by proper conversion of RSSI values (v0.3.31, v1.7.3)
   - Added fallback mechanisms for missing button.bluetooth_scan entity (v0.3.31, v1.7.3)
   - Fixed template syntax error in combined dashboard by placing JavaScript code on a single line (v0.3.32, v1.7.4)
   - Standardized default MQTT topic to "xbg" for all add-on configurations (v0.3.33, v1.7.5)
   - Fixed template syntax errors in all dashboard files by placing JavaScript code on single lines (v0.3.34, v1.7.6)

### Implementation Details:
1. In `__init__.py` (v0.3.0):
   - Implemented global reconnect state tracking at domain level
   - Added 30-second cooldown between reconnect attempts
   - Ensured proper cleanup of reconnect state even on error
   - Fixed critical issue with _async_on_advertisement call parameter mismatch
   - Added required monotonic timestamp and details parameters to advertisement calls
   - Implemented fallback MQTT message handling to prevent errors during reconnect
   - Enhanced the JSON parsing with more verbose logging
   - Created a persistent static MQTT handler to prevent multiple subscriptions
   - Improved the locking mechanism for MQTT reconnection
   - Added detailed device data format validation and logging
   - Implemented enhanced error handling in all critical sections
   - Added unsubscribe delay to ensure clean MQTT reconnects
   - Fixed potential race conditions in MQTT subscription handling

2. In `ble_scripts.yaml` (v1.4.1):
   - Removed the automatic `homeassistant.restart` service call from the `add_ble_device` script
   - Updated notification to inform users that a manual restart may be needed
   - This prevents unintended Home Assistant restarts when using the reconnect button

3. Debugging Improvements (v0.3.0):
   - Added domain-level logging of reconnect operations
   - Enhanced parameter validation in reconnect function
   - Added detailed payload structure logging for troubleshooting
   - Enhanced logging of device data format detection
   - Improved metadata extraction and device mapping
   - Added MQTT topic storage for easier diagnostics
   - Implemented better error messages throughout the codebase
   - Added defensive error handling for direct MQTT message processing

4. Dashboard Implementation (v0.3.5):
   - Created `btle_combined_dashboard.yaml` with integrated functionality
   - Combined device discovery and gateway management interfaces
   - Added tab-based navigation using Mushroom cards
   - Ensured compatibility with different entity availability states
   - Added conditional display based on entity availability

5. Enhanced BLE Discovery Add-on (v0.3.19-22, v1.6.2-4):
   - Fixed critical issue where add-on reported "Total discovered devices: 0" despite valid MQTT payload
   - Enhanced `get_ble_gateway_data()` function to detect and handle AprilBrother Gateway format
   - Improved `process_ble_gateway_data()` function to properly extract device information from nested arrays
   - Added comprehensive handling of the specific Gateway format with devices field containing array of arrays
   - Added proper creation of device entries from extracted Gateway data
   - Implemented more robust logging to show exactly what format is being detected and processed
   - Added signature pattern recognition to identify AprilBrother Gateway payloads with fields like v, mid, time, ip, mac, devices
   - Enhanced MQTT sensor discovery to find sensors containing Gateway data in different forms
   - Added comprehensive scanning of all sensor entities regardless of naming convention (v1.6.3)
   - Added detailed debugging output showing format of device data found in sensors (v1.6.3)
   - Improved entity search to scan any sensor entity that might contain device data in its state (v1.6.3)
   - Added direct MQTT sensor scanning that analyzes the first 20 MQTT sensors for device data (v1.6.3)
   - Enhanced combined dashboard with improved device selection functionality (v0.3.22, v1.6.4)
   - Added visual RSSI signal strength indicators and explanation (v0.3.22, v1.6.4)
   - Improved gateway status display with detailed information (v0.3.22, v1.6.4)
   - Fixed broken device selection in discovery view (v0.3.22, v1.6.4)
   - Added comprehensive help and troubleshooting information (v0.3.22, v1.6.4)
   - Fixed template syntax errors in device selection code (v0.3.23, v1.6.5)

---
> Source: [festion/hass-ab-ble-gateway-suite](https://github.com/festion/hass-ab-ble-gateway-suite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
