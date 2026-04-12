## frontier

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**HexageonUnity** is a sophisticated Unity-based geological simulation that models tectonic plate movement, climate systems, and environmental processes on a hexagonal grid. The project simulates complex interactions between geology, weather, water systems, and biomes to create realistic world generation.

## Core Architecture

### Central Controller System
- **HexGrid.cs**: Main simulation orchestrator managing grid, user input, controls, and phase execution
- **Hexagon.cs**: Individual cell containing 30+ properties (altitude, temperature, wind, rainfall, magma, biomes, etc.)
- **GeoPhase.cs**: Simulation phase sequencer ensuring correct execution order and data dependencies
- **ViewManager.cs**: Modern view system handling display modes and rendering coordination

### Geological Simulation Pipeline
```
MagmaImpact → Slump → SeaLevel → ClimateTemperature → WindEffect → RiverFlow → SetBiomes → Display Refresh
```

### Key Subsystems

**Environmental Systems:**
- **WindEffect.cs**: Complex wind pattern calculations with eastern ocean boost, altitude effects, and temperature-based moisture gradients
- **RiverFlow.cs**: Dynamic river formation and water routing across terrain
- **ClimateTemperature.cs**: Randomized Arctic/Desert placement with latitude-based temperature modeling
- **SeaLevel.cs**: Dynamic sea level management affecting biomes and land visibility

**Visualization Systems:**
- **ViewManager.cs**: Central view controller with plugin architecture for different display modes
- **BiomeView.cs, MagmaView.cs, AltitudeView.cs, WeatherView.cs, TemperatureView.cs**: Specialized rendering for each overlay
- **BiomeLoader.cs**: JSON-driven biome system with environmental constraints
- **HexGridColours.cs**: Advanced color mapping for multiple data visualization modes

**Rendering Infrastructure:**
- **HexagonRenderer.cs**: Core hexagon display logic with sprite and color support
- **OverlayRenderer.cs**: Arrow and overlay rendering for wind, altitude, and magma visualization
- **River Rendering**: LineRenderer-based river network visualization with proper sorting layers

## Unity Project Structure

- **Unity Version**: 6000.1.12f1 (Unity 6)
- **Main Scene**: Assets/Scenes/SampleScene.unity
- **Core Data**: Assets/Data/biomes.json (40+ biome definitions with environmental parameters)
- **Visual Assets**: Assets/Resources/Tiles/ (200+ biome sprites)

## Development Workflow

### Building & Running
- Open project in Unity Editor 6000.1.12f1
- Use Unity's standard build process (File → Build Settings)
- Test in Play Mode within Unity Editor for simulation development

### Simulation Controls
- **Space Key**: Execute 5 geological simulation cycles
- **R Key**: Regenerate entire map (resets Era to 0, new climate assignment)
- **Number Keys 1-5**: Toggle visualization overlays:
  - 1: Biome, 2: Magma, 3: Altitude, 4: Weather, 5: Temperature
- **Mouse Interaction**: Hexagon tooltips and inspection

### Performance Monitoring
- Built-in stopwatch timing for geological phase execution
- Performance data logged to Unity Console
- InfoPanel displays Era count, average height, rainfall, and temperature (land-only metrics)

## Critical Implementation Details

### Hexagonal Grid System
- **Grid Wrapping**: Horizontal wrapping with modulo arithmetic for world-like topology
- **Neighbor Calculation**: Sophisticated 6-neighbor system with offset coordinate conversion
- **Coordinate System**: X-axis runs West (0) to East (max), Y-axis runs North (0) to South (max)

### Advanced Geological Processes
1. **MagmaImpact**: Directional tectonic movement affecting neighbors based on plate intensity/direction
2. **Slump**: Height redistribution processed from highest to lowest altitude
3. **WindEffect**: Multi-factor wind calculations with special eastern ocean moisture system
4. **RiverFlow**: Water routing creating realistic river networks with LineRenderer visualization

### Climate & Weather Systems
- **Arctic/Desert Assignment**: Randomized once at map generation, fixed throughout geological cycles
- **Eastern Ocean Boost**: 3 easternmost columns have enhanced evaporation (60f) and wind intensity (100f)
- **Temperature-Based Moisture**: WaterVapour varies by climate zone (Arctic: 20-50, Temperate: 80-100, Desert: 0-20)
- **Wind Propagation**: WindChange values propagate westward from eastern ocean through WindSources system

### Modern View Architecture
- **Plugin System**: IHexagonView interface allows easy addition of new visualization modes
- **Rendering Separation**: HexagonRenderer handles base display, OverlayRenderer handles arrows/overlays
- **River Integration**: Rivers render in Biome, Weather, and Temperature views using LineRenderer components
- **Display Refresh**: Automatic cleanup and redraw on view changes or simulation updates

### Data-Driven Configuration
- **Biomes**: JSON configuration with environmental constraints and sprite references
- **Tectonic Plates**: Dynamic generation with random size, direction, and intensity
- **Climate Parameters**: Configurable sea level (default 10000), arctic placement, wind systems

## Key Dependencies
- **Unity Packages**: 2D features, TextMeshPro (UI display)
- **Required Components**: TextMeshPro for InfoPanel display system

## Important Coordinate Conventions
- **X-axis**: West (0) to East (Width-1)
- **Y-axis**: North (0) to South (Height-1)  
- **Eastern Ocean**: PositionX >= Width-3 (3 easternmost columns)
- **Land Filter**: AltitudeVsSeaLevel > 0 for land-only calculations

## Known Implementation Notes
- **Era Counter**: Tracks total geological cycles including initial 25 generation cycles
- **Arctic Consistency**: Climate assignment (IsNorthArctic) set once in HexGrid.Start(), used throughout
- **River Cleanup**: ViewManager destroys all "RiverLine" GameObjects before each refresh
- **Performance**: Display only refreshes on final cycle of each Space press (cycle 4 of 5)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TamaEaston)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TamaEaston)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
