## dxstarter

> This is a DirectX 12 graphics engine starter project built with C++ and CMake. The project uses a custom build system with PowerShell scripts for automation.

# Cursor Rules for DXStarter Project

## Project Context
This is a DirectX 12 graphics engine starter project built with C++ and CMake. The project uses a custom build system with PowerShell scripts for automation.

## Critical Instructions

### ALWAYS Check SKILLS.md First
Before performing any build, clean, or project management tasks, ALWAYS read `.cursor/SKILLS.md` first. This file contains:
- Project-specific build commands
- Build system usage guidelines
- Configuration options
- Best practices for this project

### Build System
- Use `keeper.ps1` script for all build operations
- Never use raw CMake commands directly
- Always clean before switching between Debug/Release builds
- The project uses vcpkg for dependency management

### Project Structure
- CMake-based build system
- DirectX 12 graphics engine
- C++ project with Windows-specific code
- Build output goes to `build/` directory
- Uses Visual Studio 2022+ generator

### Code Quality Guidelines
Follow the Code Quality Guidelines defined in `.cursor/BUGBOT.md`

### File Organization
- Source code in `src/` directory
- Build scripts in `Housekeeper/` directory

## Workflow
1. **ALWAYS** read `.cursor/SKILLS.md` before starting any task
2. Use project-specific commands from SKILLS.md
3. Follow the build system guidelines
4. Check for proper error handling in DirectX code
5. Ensure resource management follows COM lifecycle rules

Remember: This project has specific build requirements and conventions. Always consult SKILLS.md for the most up-to-date project-specific instructions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dimatcimbal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
