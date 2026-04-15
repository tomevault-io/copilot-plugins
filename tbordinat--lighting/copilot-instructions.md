## lighting

> QLC+ is a professional lighting control project using Q Light Controller Plus (QLC+) software for live music performances. The project contains show files, fixture definitions, and MIDI controller configurations for various venues and events.

# QLC+ Lighting Software Project Context

## Project Overview
QLC+ is a professional lighting control project using Q Light Controller Plus (QLC+) software for live music performances. The project contains show files, fixture definitions, and MIDI controller configurations for various venues and events.

## File Types and Extensions

### Show Files (.qxw)
- **QLC+ Workspace files** - Main project files containing complete show configurations
- Contains fixtures, functions, virtual console layouts, and input/output mappings
- Located in venue-specific directories (yurodivy/, itawak/, other/)
- Naming convention: YYYYMMDD-venue-name.qxw

### Fixture Definition Files (.qxf)
- **QLC+ Fixture Definition files** - XML definitions for lighting fixtures
- Located in fixtures/ directory
- Define channel mappings, modes, and physical properties
- Follow QLC+ fixture definition schema

## Project Structure

### Venue Directories
- **yurodivy/** - Band-specific shows and performances
- **itawak/** - Label events and festivals
- **other/** - Miscellaneous venues and events
- **fixtures/** - Reusable fixture definitions
- **berhinger-bcf2000/** - MIDI controller configurations

## QLC+ XML Structure

### Common XML Elements
- `<Workspace>` - Root element for show files
- `<Fixture>` - Individual lighting fixture definitions
- `<Function>` - Scenes, chasers, collections, etc.
- `<VirtualConsole>` - User interface layout
- `<InputOutputMap>` - Hardware I/O configuration

### Fixture Configuration
- **DMX Address** - Starting channel number
- **Channel Count** - Number of DMX channels used
- **Universe** - DMX universe assignment
- **Mode** - Fixture operating mode

### Function Types
- **Scenes** - Static lighting states
- **Chasers** - Sequential lighting sequences
- **Collections** - Grouped functions
- **EFX** - Effects and animations
- **RGB Matrix** - LED panel control

## Common Patterns

### Fixture Naming
- Descriptive names with location identifiers
- Example: "SuperKolor #1" (first SuperKolor fixture, from back to front, left to right)

### DMX Addressing
- Strategic channel allocation to avoid conflicts

### Show Organization
- Date-based naming for easy identification
- Venue-specific configurations
- Reusable fixture definitions across shows

## Development Guidelines

### When Editing Show Files
- Always backup before major changes
- Test DMX addressing for conflicts
- Verify fixture mode compatibility
- Check universe assignments

### When Creating Fixture Definitions
- Follow QLC+ schema standards
- Include proper preset mappings
- Define multiple modes if applicable
- Add physical properties when known

## Documentation Reference
- **QLC+ Official Docs**: https://docs.qlcplus.org/v4
- **Version**: QLC+ 4.14.0 (current project version)
- **Author**: Thomas Bordinat (project maintainer)

## Common Tasks
- Creating new show files for venues
- Adding/removing fixtures from shows
- Programming lighting sequences
- Managing fixture definitions
- Troubleshooting DMX addressing issues

This context helps Cursor understand the specialized nature of lighting control software and provide appropriate assistance for QLC+ XML files, DMX addressing, and show programming workflows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tbordinat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
