## juxta5-nrf

> The project is organized into distinct sections:

# Project Structure

The project is organized into distinct sections:

1. Applications (`/applications/`):
   - Contains main application code
   - Each subfolder is a separate application
   - Reference these explicitly when working

2. Custom Boards (`/boards/`):
   - Contains custom board definitions
   - Located under NeurotechHub vendor

3. Libraries (`/lib/`):
   - `juxta_fram`: Hardware layer for MB85RS1M FRAM chip
   - `juxta_framfs`: File system layer for FRAM
   - `juxta_vitals_nrf52`: Core utilities for voltage/RTC
   - `lisd2h12`: Accelerometer driver

4. Samples (`/samples/`):
   - Reference implementations
   - Must be consulted before implementing new features
   - Contains examples for BLE, sensors, and basic functionality
   - Always cite the example you are using (if the example is relevant)

---

# Configuration File Standards

## Project Config (`prj.conf`)
Project configuration files should contain:
- Library enablement
- Sensor subsystem configs
- Advanced peripheral settings
- Logging configuration
- Memory and thread settings
- Application-specific overrides

## README Standards (`README.md`)
README files should focus on high-level usage and implementation guidance:

**AVOID**:
- ❌ Function-by-function documentation (belongs in .h files)
- ❌ Feature lists or bullet points
- ❌ Emojis or decorative elements
- ❌ Verbose explanations of obvious concepts
- ❌ Implementation details that don't help usage
- ❌ Building and programming instructions
- ❌ Troubleshooting sections
- ❌ Hardware verification procedures
- ❌ Usage notes and key concepts
- ❌ Support sections

**INCLUDE**:
- ✅ Concise purpose statement
- ✅ Board overview and capabilities
- ✅ Explanation of main loops, timers, and program flow
- ✅ Complete pin assignment table
- ✅ Essential usage examples

## Key Principles
1. Board defconfigs define hardware capabilities
2. Project configs define software behavior
3. READMEs guide implementation and usage
4. No duplication between files
5. Hardware settings belong in defconfig
6. Application settings belong in prj.conf
7. Function documentation belongs in .h files

## Migration Guidelines
When moving settings between files:
1. Hardware-related configs → defconfig
2. Software/application configs → prj.conf
3. Remove duplicates, keeping in defconfig
4. Document hardware limitations in defconfig
5. Move detailed function docs to header files
description:
globs:
alwaysApply: true
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Neurotech-Hub) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
