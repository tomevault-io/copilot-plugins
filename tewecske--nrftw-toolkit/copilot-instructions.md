## nrftw-toolkit

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Scala.js + Laminar** web application toolkit for the game "No Rest for the Wicked" (NRFTW). It provides an item configurator interface allowing users to build and customize equipment including weapons, armor, rings, gems, runes, and enchantments.

## Development Commands

### Initial Setup
```bash
npm install  # Install JS dependencies (one-time setup)
```

### Development Workflow
**Terminal 1 - Scala.js compilation:**
```bash
sbt ~fastLinkJS  # Incremental Scala.js compilation (keep running)
# OR start sbt first, then run: ~fastLinkJS
```

**Terminal 2 - Dev server:**
```bash
npm run dev  # Start Vite dev server on port 3333 (keep running)
```

**Access the app:** http://localhost:3333

### Build Commands
```bash
npm run build        # Production build
npm run build-debug  # Debug build
npm run preview      # Preview production build
```

### SBT Commands
```bash
sbt reload    # Reload after changes to build.sbt or plugins.sbt
sbt compile   # Compile Scala code
sbt exit      # Exit sbt session
```

## Architecture

### Tech Stack
- **Frontend:** Scala.js 3.5.2 with Laminar 17.2.0 (functional reactive UI)
- **Build:** SBT + Vite with custom plugins for Scala.js integration
- **Styling:** Less CSS with CSS imports via JSImport
- **State Management:** Laminar Vars and Signals with reactive programming

### Key Components

**Main Application (`Main.scala`):**
- Central state management with `FullState` case class
- URL-based state persistence using query parameters
- Grid-based layout with modular item builders

**Item Builders:**
- `ItemBuilder` - Handles armor pieces (helmet, armor, pants, gloves)
- `WeaponBuilder` - Handles weapons, shields, and bows with rune slots
- `RingBuilder` - Handles ring selection and management
- `GemsBuilder`, `RunesBuilder`, `EnchantmentsBuilder` - Specialized builders

**State Management:**
- `ItemState` - Core item configuration (rarity, enchants, gems)
- `WeaponState` - Extends ItemState with weapon-specific data and runes
- `FullState` - Application-wide state with URL serialization

**Data Models:**
- Game item models defined in `ItemModel.scala` with data type definitions
- Game item data organized in `items/` package with specialized files:
  - `Equipment.scala` - Weapons, armor, and equipment data
  - `Gems.scala` - Gem definitions and properties
  - `Rings.scala` - Ring types and attributes
  - `Runes.scala` - Rune data and effects
- Image assets organized by type (gems, rings, runes) in `public/images/`

### Key Patterns

**Reactive State:**
- Uses Laminar's `Var` for mutable state and `Signal` for reactive updates
- State zooming with `zoomLazy` for component isolation
- Automatic URL updates via `replaceState` on state changes

**CSS Integration:**
- Each component imports its CSS via `@JSImport` annotations
- Less files processed through Vite build pipeline
- Modular styling with component-specific CSS files

**Error Handling:**
- Centralized error validation in `Errors.scala`
- Error states tracked alongside data states
- Visual error indicators in UI components

### File Organization
```
src/main/scala/com/tewe/nrftw/
├── Main.scala              # Application entry point
├── ItemModel.scala         # Game data type definitions
├── ItemState.scala         # State case classes
├── ItemBuilder.scala       # Armor item builder
├── WeaponBuilder.scala     # Weapon/bow/shield builder
├── RingBuilder.scala       # Ring selection builder
├── *Builder.scala          # Specialized component builders
├── Modal.scala             # Modal dialog component
├── Errors.scala            # Validation logic
├── items/                  # Game data package
│   ├── Equipment.scala     # Weapons, armor, equipment data
│   ├── Gems.scala          # Gem definitions
│   ├── Rings.scala         # Ring data
│   └── Runes.scala         # Rune data
└── *.css                   # Component stylesheets
```

## Development Notes

### Adding Dependencies
- Scala dependencies: Add to `libraryDependencies` in `build.sbt` using `%%%` (not `%%`)
- JS dependencies: `npm i package-name`
- Dev dependencies: `npm i --include=dev package-name`

### State Serialization
- Application state is URL-encoded for sharing builds
- Each component implements `shortState()` for serialization
- State restoration from URL parameters handled in `Main.scala`

### Scala.js Specifics
- Uses ES Module output (`ModuleKind.ESModule`)
- Main module initializer enabled
- DOM manipulation via `scalajs-dom` library
- Vite handles Scala.js integration via `@scala-js/vite-plugin-scalajs`

---
> Source: [tewecske/nrftw-toolkit](https://github.com/tewecske/nrftw-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
