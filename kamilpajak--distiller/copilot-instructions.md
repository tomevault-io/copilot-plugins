## distiller

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

### Building the Project

```bash
# Build for the Arduino MKR WiFi 1010
pio run

# Build for a specific environment (prod, native, test, ci)
pio run -e prod   # Production build for Arduino MKR WiFi 1010
pio run -e native # Native build for development
pio run -e test   # Test build
pio run -e ci     # CI build with warnings as errors

# Upload to Arduino MKR WiFi 1010
pio run -t upload

# Clean build
pio run -t clean
```

### Testing

```bash
# Run all tests using Docker (recommended for consistent environment)
./scripts/pio-tools.sh test

# Alternative: Run tests directly with PlatformIO
pio test -e test

# Run a specific test file
pio test -e test -f test_thermometer
```

### Code Quality Tools

```bash
# Format code using clang-format
./scripts/pio-tools.sh format

# Run static analysis using clang-tidy
./scripts/pio-tools.sh tidy

# Run all checks (format, tidy, test) sequentially
./scripts/pio-tools.sh all
```

### Docker Development Environment

The project uses a Docker container for development tasks. The `pio-tools.sh` script simplifies running commands:

```bash
# Build Docker image (if not already built)
docker build -t distiller-tools .

# Run a command in the Docker container
docker run -v $(pwd):/project distiller-tools [command]
# Where [command] can be: format, tidy, test, all
```

## System Architecture

The Distiller is an automated distillation control system built for the Arduino MKR WiFi 1010. It follows a modular architecture with clear separation of concerns.

### Core Components

1. **Main Controller** (main.cpp): Orchestrates the overall distillation process
2. **Distillation State Manager**: Manages the distillation process state transitions
3. **Heater Controller**: Controls heating elements at 3 power levels
4. **Valve Controller**: Manages 8 valves for different fractions and coolant
5. **Thermometer Controller**: Manages 4 temperature sensors
6. **Scale Controller**: Manages 6 weight sensors
7. **Flow Controller**: Controls flow rates using PID
8. **Display Controller**: Manages the LCD display

### Hardware Abstraction Layer

1. **Relay**: Abstracts digital output pins for heaters and valves
2. **Thermometer**: Abstracts DS18B20 temperature sensors
3. **Scale**: Abstracts HX711 load cell amplifiers
4. **LCD**: Abstracts I2C LCD display

### Key Design Patterns

1. **Singleton Pattern**: Used for DistillationStateManager to ensure a single source of truth
2. **State Pattern**: The distillation process is managed through a series of states
3. **Task Scheduling**: Uses TaskManagerIO for non-blocking concurrent operations
4. **PID Control**: Maintains consistent flow rates by controlling valve openings
5. **Adapter/Delegation Pattern**: Used for mocking in tests

### Distillation Process Flow

The system progresses through these states automatically:
1. HEAT_UP → STABILIZING (when temperature > 40°C)
2. STABILIZING → EARLY_FORESHOTS (when temperature stabilizes)
3. EARLY_FORESHOTS → LATE_FORESHOTS (after 200ml)
4. LATE_FORESHOTS → HEADS (after 400ml)
5. HEADS → HEARTS (after 900ml)
6. HEARTS → EARLY_TAILS (after 5000ml or temperature increase)
7. EARLY_TAILS → LATE_TAILS (after 700ml)
8. LATE_TAILS → FINALIZING (after 600ml)
9. FINALIZING → OFF (after 10 minutes)

## Testing Strategy

The project uses Google Test and Google Mock for unit testing. Tests are designed to run on the development machine, not on the Arduino hardware.

### Mock Arduino Implementation

The project includes a mock Arduino implementation (`mock_arduino.h` and `mock_arduino.cpp`) that provides:
- Mock implementations of Arduino functions (pinMode, digitalWrite, millis)
- Helper functions for setting and advancing mock time
- Test fixtures for pin operations

### Unit Test Structure

Tests use conditional compilation (#ifdef UNIT_TEST) to adapt production code for testing:
- Controller classes often have separate constructors for test and production environments
- Hardware-dependent code is abstracted and mockable
- Google Mock allows verification of hardware interactions

### Testing Command Example

To run a specific test:
```bash
pio test -e test -f test_thermometer
```

## Library Dependencies

The project uses the following Arduino libraries:
- HX711 (v0.7.5): For load cell amplifiers
- DallasTemperature (v3.11.0): For temperature sensors
- PID (v1.2.1): For implementing PID control
- hd44780 (v1.3.2): For LCD display
- TaskManagerIO (v1.4.3): For task scheduling
- OneWire: For OneWire devices
- Wire: For I2C communication

For testing, it uses:
- Google Test/Mock (v1.15.2): For unit testing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kamilpajak)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kamilpajak)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
