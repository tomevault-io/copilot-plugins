## claude-deck

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Deck is a Stream Deck plugin that provides visual feedback for Claude Code tasks. It allows reserving Stream Deck keys to show task status (idle, running, completed) with project names displayed on each button, and provides an HTTP API for Claude Code to interact with these visual slots.

## Development Commands

- **Build plugin**: `npm run build` - Compiles TypeScript to JavaScript using Rollup
- **Development watch**: `npm run watch` - Builds with file watching and automatically restarts the Stream Deck plugin
- **Test scripts**: 
  - `./scripts/test/test-all-slots.sh` (Linux/Mac) or `./scripts/test/test-all-slots.bat` (Windows) - Tests multiple slots
  - `./scripts/test/test-overflow.sh` (Linux/Mac) or `./scripts/test/test-overflow.bat` (Windows) - Tests slot overflow behavior
- **Generate SVGs**: `python scripts/generate_progress_svgs.py` - Generates animation frames for pulsating Claude icon in running state

## Architecture

### Core Components

- **Stream Deck Plugin** (`src/plugin.ts`): Main plugin code that handles Stream Deck integration
- **HTTP Server**: Express server (default port 17880) that provides REST API for Claude Code tasks
- **Task Management**: In-memory registry tracking Claude Code tasks and their assigned slots
- **Visual Rendering**: SVG-based rendering system showing Claude icon in different states (grey idle, pulsating running, static colored completed) with project name overlay

### Key Concepts

- **Tasks**: Claude Code user requests that can be tracked (states: idle, running, completed)
- **Slots**: Individual Stream Deck keys that display task status and project name
- **Task Pool**: Automatic assignment of tasks to available slots

### HTTP API Endpoints

- `GET /slots` - Get slot status and availability
- `POST /start` - Start a task and assign to slot
- `POST /finish` - Mark task as completed

### Build System

- **TypeScript**: Source code in `src/` directory
- **Rollup**: Bundles to `pro.clever.claudedeck.sdPlugin/bin/plugin.cjs`
- **Stream Deck SDK**: Uses `@elgato/streamdeck` for plugin integration
- **Dependencies**: Express for HTTP server, Zod for validation, built-in animation system

### Plugin Structure

- `manifest.json`: Stream Deck plugin metadata and configuration
- Action UUID: `pro.clever.claudedeck.slot` for task slot keys

## Key Implementation Details

- Tasks are automatically assigned to free slots when started
- Slots show visual feedback with Claude icon and project name: grey when idle, pulsating in Claude colors when running, static in Claude colors when completed
- Users can tap completed slots to clear them and free up space for new tasks
- HTTP port is configurable via the property inspector (defaults to 17880)
- Plugin persists through Stream Deck restarts and reconnects tasks to their slots
- Running tasks display pulsating Claude icon animation (8 frames, 125ms per frame) - icon scales in size with Claude Anthropic brand colors
- Task state management is handled in-memory with automatic cleanup

## Code Architecture

### Core Files

- `src/plugin.ts` - Stream Deck plugin entry point, handles slot lifecycle and user interactions
- `src/http-server.ts` - Express server providing REST API endpoints for external services
- `src/services.ts` - Task registry and slot assignment logic with in-memory state management
- `src/renderer.ts` - SVG-based visual rendering for different task states
- `src/types.ts` - TypeScript type definitions for Task and Slot interfaces
- `src/config.ts` - Configuration constants (HTTP port, animation settings)

### Data Flow

1. Claude Code calls `POST /start` with task ID and project name
2. Task gets assigned to free slot via `assignTaskToSlot()`
3. Slot renders running state with pulsating Claude icon and displays project name
4. Claude Code calls `POST /finish` with completion status
5. Slot renders final state (completed) with static Claude icon and project name
6. User can tap slot to clear and return to free pool for new tasks

---
> Source: [etechlead/claude-deck](https://github.com/etechlead/claude-deck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
