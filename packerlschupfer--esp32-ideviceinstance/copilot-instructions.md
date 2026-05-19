## esp32-ideviceinstance

> Abstract interface for device instances in ESP32/FreeRTOS environment. Defines standardized contract for Modbus devices, sensors, and other peripherals.

# IDeviceInstance - CLAUDE.md

## Overview
Abstract interface for device instances in ESP32/FreeRTOS environment. Defines standardized contract for Modbus devices, sensors, and other peripherals.

## Key Features
- Abstract base class for device implementations
- Unified error codes (`DeviceError`)
- Result type using `common::Result<T, DeviceError>`
- Thread-safe access patterns
- Event group integration
- Extensible data types

## Interface Methods

### Lifecycle
- `initialize()` - Initialize device hardware
- `waitForInitializationComplete(timeout)` - Block until ready

### Data Operations
- `requestData()` - Request data from device
- `processData()` - Process received data
- `getData(dataType)` - Get data by type
- `waitForData()` - Block until data available

### Actions
- `performAction(actionId, param)` - Execute device-specific action

### Thread Safety
- `getMutexInstance()` - Get instance mutex
- `getMutexInterface()` - Get interface mutex
- `getEventGroup()` - Get event group for synchronization

## Error Codes
```cpp
enum class DeviceError {
    SUCCESS,
    NOT_INITIALIZED,
    TIMEOUT,
    MUTEX_ERROR,
    COMMUNICATION_ERROR,
    INVALID_PARAMETER,
    DATA_NOT_READY,
    MEMORY_ERROR,
    DEVICE_BUSY,
    NOT_SUPPORTED,
    UNKNOWN_ERROR
};
```

## Implementing Classes
- `RYN4` - Relay module
- `MB8ART` - Temperature sensor module
- Both inherit from `QueuedModbusDevice` and `IDeviceInstance`

## Usage Pattern
```cpp
class MyDevice : public IDeviceInstance {
    DeviceResult<void> initialize() override {
        // Init hardware
        return DeviceResult<void>::ok();
    }

    DeviceResult<std::vector<float>> getData(DeviceDataType type) override {
        // Return sensor data
    }
};
```

## Thread Safety
All implementations must ensure thread safety for public methods using mutexes and atomics.

---
> Source: [packerlschupfer/ESP32-IDeviceInstance](https://github.com/packerlschupfer/ESP32-IDeviceInstance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
