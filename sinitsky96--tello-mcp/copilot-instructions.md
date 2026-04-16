## tello-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Tello MCP (Model Context Protocol) Server that enables Claude Desktop to control DJI Tello drones through natural language commands. 

**Important Architecture Update**: The MCP server now controls the drone through the GUI interface instead of directly using the Tello API. This prevents connection conflicts since the Tello drone can only maintain one connection at a time. The MCP server has full visibility and control over the GUI, including screenshot capabilities and access to all visual elements.

## Common Development Commands

### Setup and Installation
```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Install in development mode
pip install -e .
```

### Running Tests
```bash
# Run all tests
python3 -m pytest tests/

# Run specific test file
python3 -m pytest tests/test_models.py

# Run with verbose output
python3 -m pytest -v tests/
```

### Running the Server
```bash
# After installation
tello-mcp

# Or directly
python3 -m src.server
```

## Code Architecture

### Core Components

1. **src/models.py**: Data structures and enums
   - `DroneStatus`: Telemetry data (battery, height, temperature, etc.)
   - `Position`: 3D coordinates and orientation
   - `Direction`, `FlipDirection`, `VideoResolution`: Control enums

2. **src/tello_api.py**: Tello SDK wrapper
   - Manages `djitellopy.Tello` instance
   - Thread-safe video streaming with OpenCV
   - Comprehensive safety checks (battery levels, connection status)
   - All methods return detailed operation results

3. **src/gui_controller.py**: GUI Control Bridge (NEW)
   - Provides programmatic control of GUI elements
   - Thread-safe command queue for async operations
   - Full GUI state inspection (screenshots, button states, telemetry)
   - Replaces direct TelloAPI usage in MCP server

4. **src/server.py**: MCP server implementation
   - Async tool registration following MCP protocol
   - Maps natural language commands to GUI controller methods
   - Error handling and response formatting
   - New GUI inspection tools (screenshot, state, telemetry)

5. **src/gui/**: GUI components
   - `main_window.py`: Main application window with MCP integration
   - `control_panel.py`: Flight controls (now externally controllable)
   - `camera_view.py`: Video display with frame access
   - `telemetry_panel.py`: Real-time telemetry with data export

### Key Design Patterns

- **GUI-Based Control**: MCP server controls drone through GUI to avoid connection conflicts
- **Full Observability**: MCP can see and control all GUI elements, including screenshots
- **Safety-First**: Every operation checks connection status and battery levels
- **Thread Safety**: Video streaming and command processing use separate threads
- **Error Handling**: All operations wrapped in try-except with descriptive error messages
- **Command Queue**: Async command processing ensures GUI responsiveness

### Important Safety Features

1. Battery checks before operations (warns if <20%, prevents critical operations if <10%)
2. Automatic landing on disconnect
3. Emergency stop functionality
4. Parameter validation for all movements
5. Connection state verification before commands

## New MCP Tools for GUI Inspection

The MCP server now includes tools for full GUI visibility:
- `get_gui_screenshot`: Capture the entire GUI window
- `get_gui_state`: Get complete state including buttons and telemetry
- `get_button_states`: Check which controls are enabled/disabled
- `get_telemetry_readings`: Read all displayed telemetry values

## Usage Notes

1. **Starting the System**: 
   - Run `python launch_gui.py` to start the GUI with WebSocket server
   - The GUI starts a WebSocket server on port 8765
   - Configure Claude Desktop to use the WebSocket-based MCP server

2. **Connection Flow**:
   - Claude → MCP Server → WebSocket → GUI → Tello API → Drone
   - WebSocket provides real-time bidirectional communication
   - Only one connection path ensures no conflicts

3. **Visual Feedback**:
   - All operations show visual feedback in the GUI
   - MCP can capture screenshots to verify operations
   - Telemetry updates in real-time

4. **WebSocket Architecture**:
   - GUI runs WebSocket server (port 8765)
   - MCP server connects as WebSocket client
   - Commands flow over WebSocket in real-time
   - Both request-response and async messaging supported

## Testing Considerations

Currently only basic model tests exist. When adding new features:
- Test GUI controller integration
- Verify command queue processing
- Test safety validations thoroughly
- Mock `djitellopy.Tello` for unit tests
- Verify thread safety for video operations
- Test error conditions and edge cases

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sinitsky96) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
