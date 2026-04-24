## idle-game

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development
- `lamdera live` - Start development server
- `npm run build:css -- --watch` - Watch CSS changes (run in separate terminal)
- `lamdera make src/Frontend.elm src/Backend.elm` - Compile and check for errors

### Testing
- `npx elm-test path/to/test.elm --watch` - Run single test file with watch mode
- `npx elm-test "src/**/*Tests.elm"` - Run all tests matching glob pattern
- `npx playwright test --reporter=line` - Run E2E tests
- `npx playwright install` - Install Playwright browsers (one-time setup)

### Performance Testing
- `npx elm-test src/IdleGame/PerfTesting/Test60Days.elm` - Run performance tests for Time Passes calculations

### Code Quality
- `npx elm-review --fix` - Run linter and auto-fix issues
- `npx elm-format` - Format Elm code

### Code Generation
- `npm run generate-elm` - Generate boilerplate code using elm-codegen
- `cd codegen && npx elm-codegen install` - Update codegen dependencies

## Architecture

This is a **Lamdera** full-stack Elm application implementing an idle game with a medical school theme. The codebase is structured as follows:

### Core Architecture
- **Frontend.elm** - Main frontend entry point
- **Backend.elm** - Server-side logic and state management
- **Types.elm** - Shared types between frontend and backend
- **Evergreen/** - Migration system for type changes across versions ⚠️ **DO NOT EDIT** - This system is not well documented and agents should not modify these files

### Game Logic (`src/IdleGame/`)
- **Game.elm** - Core game state and logic
- **Activity.elm** - Activity system (studying subjects)
- **Timer.elm** - Time-based progression mechanics
- **Resource.elm** - Resource management (knowledge points)
- **Xp.elm** - Experience point system
- **Coin.elm** - Currency system
- **ShopUpgrade.elm** - Shop items and upgrades
- **Skill.elm** - Skill system (medical subjects)
- **Views/** - UI components for game elements

### Code Generation (`codegen/`)
Uses elm-codegen to generate boilerplate. The generated code appears in both `codegen/Gen/` and `src/` directories. Edit `codegen/Generate.elm` to modify generation logic.

### Testing Strategy
- Unit tests for core game logic (Timer, Xp, Game modules)
- Performance tests for Time Passes calculations
- E2E tests with Playwright for user workflows
- Run E2E tests only when behavioral changes are complete, not during iterations

### Key Patterns
- **Effect system** - Custom effects for game actions
- **Time simulation** - Calculate progression when player returns after time away
- **Resource constraints** - Activities require specific resources to proceed
- **Mastery system** - Progressive unlocking of content through skill advancement

The game uses Tailwind CSS with DaisyUI for styling and includes a comprehensive deployment checklist in the README.

## Type System Conventions

- We don't tend to have "id" fields on our types in this codebase. Instead there's a union type for e.g. ResourceKind = Wood | Stone where the type Wood signifies wood, rather than resource of id resource_1 or something.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tristanpendergrass) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
