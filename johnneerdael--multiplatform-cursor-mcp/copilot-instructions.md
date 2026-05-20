## multiplatform-cursor-mcp

> 1. Windows-specific Path Handling

# Cross-Platform Migration Task

## Issues Found
1. Windows-specific Path Handling
   - Using `process.env.LOCALAPPDATA` for Cursor.exe path
   - Windows-style path separators in path.join operations
   - Hard-coded Windows paths in documentation

2. Windows-specific Dependencies
   - `node-windows` dependency for window management
   - Heavy reliance on `node-window-manager` for Windows-specific window operations
   - `robotjs` for Windows-specific keyboard/mouse automation

3. Windows-specific APIs
   - Windows-specific window handling in WindowsApiService
   - Windows-specific process management
   - Windows-specific keyboard event handling

## Migration Plan
[X] 1. Path Handling
    - Replace Windows-specific path handling with path.join and process.platform checks
    - Update documentation with cross-platform examples
    - Create platform-specific path resolvers

[X] 2. Window Management
    - Create platform-agnostic window management interface
    - Implement platform-specific adapters (Windows/MacOS/Linux)
    - Replace direct Windows API calls with adapter interface

[X] 3. Process Management
    - Create cross-platform process spawning logic
    - Handle platform-specific executable paths
    - Implement platform-specific process detection

[X] 4. Keyboard/Mouse Automation
    - Create platform-agnostic input automation interface
    - Replace robotjs with @nut-tree/nut-js
    - Implement platform-specific input handling

[X] 5. Configuration
    - Add platform-specific configuration options
    - Create default configs for each supported platform
    - Update environment variable handling

## Completed Tasks
1. Created PathResolver utility for cross-platform path handling
2. Created platform-agnostic window management interface
3. Implemented Windows and macOS window managers
4. Implemented Linux window manager using xdotool
5. Created cross-platform input automation service using @nut-tree/nut-js
6. Updated CursorInstanceManager to use new abstractions
7. Updated documentation with cross-platform instructions
8. Added comprehensive test suite for window managers
9. Added structured error handling system
10. Added logging system with context tracking
11. Added tests for input automation service
12. Added tests for path resolver
13. Created platform-specific installation scripts
14. Added integration tests for window management
15. Added integration tests for input automation
16. Added GitHub Actions workflows for testing and releases

## Next Steps
1. Add performance monitoring and metrics
2. Create platform-specific uninstallation scripts

## Priority Files to Update
1. ~~src/services/WindowsApiService.ts~~ (Replaced with new window manager)
2. ~~src/managers/CursorInstanceManager.ts~~ (Updated)
3. ~~src/types/node-windows.d.ts~~ (No longer needed)
4. ~~src/test/* files~~ (Added comprehensive tests)
5. ~~README.md~~ (Updated with cross-platform instructions)

## Dependencies
1. Added:
   - @nut-tree/nut-js (cross-platform input automation)
   - active-win (cross-platform window detection)
   - winston (logging)

2. Removed:
   - robotjs (Windows-specific)
   - node-windows (Windows-specific)
   - node-window-manager (Windows-specific)

## Platform Support
1. Windows:
   - Uses native window management through active-win
   - Input automation through @nut-tree/nut-js
   - PowerShell installation script with scheduled task

2. macOS:
   - Uses AppleScript for window management
   - Input automation through @nut-tree/nut-js
   - Requires accessibility permissions
   - Launch agent for auto-start

3. Linux:
   - Uses xdotool for window management
   - Input automation through @nut-tree/nut-js
   - Requires X11 and xdotool
   - Systemd service for auto-start
   - udev rules for input devices

## Testing
1. Unit Tests:
   - Window manager tests for each platform
   - Error handling tests
   - Input automation tests
   - Path resolver tests

2. Integration Tests:
   - Cross-platform window management
   - Input automation scenarios
   - Error recovery scenarios

3. Platform-Specific Tests:
   - Windows: Native window handling
   - macOS: AppleScript integration
   - Linux: xdotool commands

4. CI/CD:
   - GitHub Actions workflow for PR tests
   - GitHub Actions workflow for releases
   - Cross-platform test matrix
   - Automated artifact uploads

## Installation
1. Windows:
   - PowerShell script with admin privileges
   - Creates scheduled task for auto-start
   - Sets up necessary directories and permissions

2. macOS:
   - Bash script with sudo
   - Creates launch agent for auto-start
   - Handles accessibility permissions
   - Sets up Xcode Command Line Tools

3. Linux:
   - Bash script with sudo
   - Supports apt, dnf, and pacman
   - Creates systemd user service
   - Sets up udev rules for input devices 

---
> Source: [johnneerdael/multiplatform-cursor-mcp](https://github.com/johnneerdael/multiplatform-cursor-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
