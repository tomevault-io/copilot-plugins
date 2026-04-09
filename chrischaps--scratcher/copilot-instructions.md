## scratcher

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Unity 6000.2.5f1 project called "Scratcher" - a 2D isometric fishing/farming game configured for development using the Universal Render Pipeline (URP). The project is now version controlled with Git and hosted on GitHub at: https://github.com/chrischaps/Scratcher

## Game Features

This project implements a 2D isometric game with the following systems:

### Core Systems
- **Terrain Generation**: Procedural terrain with grass and water tiles using Unity's Tilemap system
- **Player Movement**: Multiple controller options including grid-based, isometric, and terrain-aware movement
- **Fishing System**: Complete fishing mechanics with water zones, fish database, and catch mechanics
- **Inventory System**: Item management and storage functionality with event-driven updates
- **Time Management**: Game time progression system with day/night cycle
- **Camera System**: Flexible camera controller for 2D isometric gameplay
- **Modern UI System**: Professional UI Toolkit-based interface with responsive design

### Implemented Components
- **Data Systems**: Game Manager, Fish Database, Level Configuration, Inventory System, Game Time Manager
- **Player Controllers**: Grid-based, Isometric, and Terrain-aware movement options
- **Terrain Management**: Terrain generation and layer management with procedural lakes
- **Tile System**: Custom tile implementations for different terrain types
- **UI Toolkit System**: Modern, performant UI with UXML layouts and USS styling
- **Debug Tools**: Comprehensive debug panel with system monitoring and cheat functions
- **Notification System**: Toast notifications with animations and sound support

## Unity Project Structure

The project follows a logical, organized folder structure:

- **Assets/Art/**: All visual assets
  - `Sprites/Characters/`: Player and NPC sprites with animations
  - `Sprites/Environment/`: Terrain, objects, and environmental sprites (including Sprout Lands assets)
  - `Sprites/UI/`: User interface graphics
  - `Materials/`: Shader materials and textures
- **Assets/Audio/**: Sound effects and music (ready for future audio assets)
- **Assets/Data/**: Game data and configurations
  - `Configurations/`: ScriptableObject game settings including LevelConfiguration
  - `ScriptableObjects/`: Data containers and game objects
- **Assets/Prefabs/**: Reusable game object prefabs
- **Assets/Scenes/**: Unity scene files (SampleScene.unity, Game.unity)
- **Assets/Scripts/**: Organized C# scripts with subdirectories:
  - `Camera/`: Camera control systems (FlexibleCameraController)
  - `Data/`: Core data management (GameManager, Fish data, Inventory, GameTimeManager, etc.)
  - `Fishing/`: Fishing gameplay mechanics (FishingController, WaterZone)
  - `Player/`: Player movement controllers (IsometricPlayerController, GridBasedPlayerController, TerrainAwarePlayerController)
  - `Terrain/`: Terrain generation and management (TerrainGenerator, TerrainLayerManager)
  - `Tiles/`: Custom tile implementations (NormalTile, WaterTile)
  - `UI/`: Legacy UI components (FishingGameUI - being replaced by UI Toolkit)
- **Assets/UI/**: Modern UI Toolkit system
  - `UXML/`: UI layout files (GameHUD.uxml, DebugPanel.uxml, MainMenu.uxml)
  - `USS/`: Stylesheets (MainHUD.uss, DebugUI.uss, MainMenu.uss)
  - `Scripts/`: UI controllers and managers
  - `Themes/`: UI theme variations
  - `Resources/`: UI-specific assets
  - `SETUP_GUIDE.md`: Comprehensive UI setup documentation
- **Assets/Settings/**: Project configuration files
  - `Input/`: Input system configurations (InputSystem_Actions.inputactions)
  - `Rendering/`: URP and rendering settings
  - `Project/`: Unity project templates and settings
- **Assets/World/**: World-building assets
  - `Tiles/`: Tile assets for terrain generation
  - `TilePalettes/`: Tile palette configurations
  - `Terrain/`: Terrain-specific assets
- **Docs/**: System documentation and guides
  - Developer guides for implementing and configuring game systems
  - **IMPORTANT**: When adding or updating game systems, create or update documentation in this folder
  - Each major system should have a comprehensive guide explaining setup, configuration, and usage
- **ProjectSettings/**: Unity project configuration files
- **Packages/**: Unity package dependencies managed via Package Manager

## Modern UI System (UI Toolkit)

The project features a comprehensive UI Toolkit implementation providing:

### UI Architecture
- **Foundation Classes**: UIToolkitPanel base class, UIManager for state management
- **Data Binding**: Reactive UI updates with DataBindingHelper utilities
- **Event System**: Integration with existing game systems through events
- **Responsive Design**: Flexbox layouts that adapt to different screen sizes

### UI Components
- **GameHUD**: Modern game interface with time display, inventory, player stats, fishing overlay
- **Debug Panel**: 5-tabbed debug interface (System Stats, Game State, Fishing, Terrain, Cheats)
- **Main Menu**: Professional menu system with settings, save/load, credits
- **Notification System**: Toast notifications with animations and type-based styling

### UI Files Structure
```
Assets/UI/
├── UXML/           # Layout documents (GameHUD.uxml, DebugPanel.uxml, MainMenu.uxml)
├── USS/            # Stylesheets (MainHUD.uss, DebugUI.uss, MainMenu.uss)
├── Scripts/        # Controllers (GameHUDController, DebugUIManager, MainMenuController, etc.)
├── Themes/         # Theme variations
└── Resources/      # UI assets
```

### Key UI Features
- **Hardware-accelerated rendering** with UI Toolkit
- **CSS-like styling** workflow with USS
- **Modular panel system** with smooth transitions
- **Debug tools** accessible via F1 key
- **Input System integration** (no legacy Input usage)
- **Automatic integration** with existing game systems

## Input System

The project uses Unity's Input System with comprehensive input action mappings configured in `Assets/Settings/Input/InputSystem_Actions.inputactions`:

### Player Actions
- **Move**: WASD/Arrow keys, gamepad left stick, touch gestures
- **Look**: Mouse delta, gamepad right stick
- **Attack**: Mouse left click, gamepad X button, touch tap, Enter key
- **Interact**: E key, gamepad Y button (hold interaction)
- **Crouch**: C key, gamepad B button
- **Jump**: Space bar, gamepad A button
- **Sprint**: Left Shift, gamepad left stick press
- **Previous/Next**: Number keys 1/2, gamepad D-pad

### UI Actions
- **F1**: Toggle debug panel
- **ESC**: Close top UI panel
- Standard UI navigation with support for all input methods (keyboard, gamepad, touch, XR)

**Important**: The project uses the NEW Input System exclusively. All UI scripts use `UnityEngine.InputSystem` instead of legacy `UnityEngine.Input`.

## Key Unity Packages

- Universal Render Pipeline (URP) 17.2.0
- Unity Input System 1.14.2
- Unity Test Framework 1.5.1
- 2D Animation, Sprites, and Tilemap tools
- Visual Scripting 1.9.7
- UI Toolkit (built-in with Unity 6000.2.5f1)

## Development Commands

### Building the Project
- Open project in Unity Editor
- File → Build Settings → Build (or Ctrl+Shift+B)
- For command line builds: Use Unity's `-batchmode` with `-buildTarget` parameter

### Testing
- Unity Test Framework is included (com.unity.test-framework 1.5.1)
- Run tests via Window → General → Test Runner in Unity Editor
- Write tests in `Assets/Tests/` directory (create if needed)

### UI Development
- **UI Builder**: Use Window → UI Toolkit → UI Builder to edit UXML files
- **UI Debugger**: Use Window → UI Toolkit → Debugger for runtime debugging
- **Hot Reload**: USS changes apply immediately in Play mode
- **Debug Panel**: Press F1 in-game to access comprehensive debug tools

## Version Control

This project is tracked with Git and hosted on GitHub:
- **Repository**: https://github.com/chrischaps/Scratcher
- **Branch**: master
- **Git Setup**: Includes Unity-specific .gitignore file
- All Unity-generated and temporary files are properly excluded from version control

## Architecture Notes

This Unity project is configured for 2D isometric game development with:
- Universal Render Pipeline for optimized 2D rendering
- Component-based architecture with modular systems
- ScriptableObject-based configuration system (LevelConfiguration, FishDatabase)
- Modern UI Toolkit for performant, maintainable UI
- Comprehensive input system supporting multiple platforms
- Event-driven architecture for system communication
- 2D-focused package selection (animation, sprites, tilemaps)
- Multi-platform input scheme (desktop, mobile, gamepad, XR ready)
- Organized code structure following Unity best practices

## Game Systems Integration

### Core System Workflow
1. **GameManager**: Central coordination point, initializes all systems
2. **UIIntegrationManager**: Connects UI Toolkit with game systems
3. **Event-driven updates**: Systems communicate through events (OnFishAdded, OnTimeChanged, etc.)
4. **Automatic UI updates**: Data binding ensures UI reflects game state changes

### Key Integration Points
- **FishingController** ↔ **GameHUD**: Real-time fishing UI updates
- **InventorySystem** ↔ **NotificationManager**: Fish catch notifications
- **GameTimeManager** ↔ **UI displays**: Time/day progression
- **Debug systems** ↔ **All game systems**: Comprehensive monitoring and control

## File Conventions

- C# scripts should be placed in `Assets/Scripts/` with appropriate subdirectories
- UI Toolkit files go in `Assets/UI/` (UXML, USS, Scripts)
- Scene files go in `Assets/Scenes/`
- Use Unity's standard naming conventions (PascalCase for classes, camelCase for variables)
- Follow Unity's component-based architecture patterns
- UI components inherit from UIToolkitPanel base class
- Use UIManager.Instance for centralized UI management

## Development Best Practices

### UI Development
- Use UI Toolkit instead of Canvas-based UI for new interfaces
- Leverage data binding for reactive UI updates
- Follow the established USS styling patterns
- Use the debug panel (F1) for testing and development
- Refer to `Assets/UI/SETUP_GUIDE.md` for detailed UI documentation

### Input Handling
- Always use UnityEngine.InputSystem, never legacy Input
- Input actions are configured in InputSystem_Actions.inputactions
- UI scripts handle input through Input System events

### System Integration
- Use events for loose coupling between systems
- Register UI panels with UIManager for centralized control
- Follow the established naming conventions for UI elements
- Leverage the NotificationManager for user feedback

### Documentation
- **IMPORTANT**: When adding or updating game systems, create or update documentation in the `Docs/` folder
- Each major system should have a comprehensive markdown guide covering:
  - System overview and architecture
  - Prerequisites and dependencies
  - Step-by-step setup instructions
  - Configuration options and best practices
  - Testing procedures
  - Troubleshooting common issues
  - Code examples and API references
- Existing guides:
  - `Docs/FISHING_ZONES_GUIDE.md` - Complete fishing system and water zone setup
  - `Docs/TERRAIN_GENERATION_GUIDE.md` - Procedural terrain generation system
  - `Docs/INVENTORY_SYSTEM_GUIDE.md` - Fish inventory and item management
  - `Docs/GAME_TIME_SYSTEM_GUIDE.md` - Day/night cycle and time-based events
  - `Docs/LEVEL_CONFIGURATION_GUIDE.md` - ScriptableObject-based level presets

## Debugging and Development Tools

### Debug Panel (F1)
- **System Tab**: Performance metrics, memory usage, FPS monitoring
- **Game State Tab**: Time controls, player info, inventory management
- **Fishing Tab**: Water zones, fish database, catch rate debugging
- **Terrain Tab**: Live terrain generation with parameter controls
- **Cheats Tab**: Developer tools (add fish, teleport, system toggles)

### UI Development
- **UI Builder**: Visual UXML editing
- **UI Debugger**: Runtime UI inspection
- **Hot Reload**: USS changes apply immediately
- **Console Integration**: Debug messages and error tracking

This project represents a modern Unity development approach combining traditional 2D game systems with cutting-edge UI technology for both player experience and developer productivity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrischaps)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chrischaps)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
