## app0

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

App0 is a desktop application built with Electron that allows users to generate and run small personal apps ("applets") using AI without coding, terminals, or installing dependencies. The application handles everything internally - app generation, installation, hosting, database setup, and launching.

## Architecture

App0 follows a modular architecture with these core components:

### Core Directory Structure
```
App0/
├── core/              # AI logic, app generator, build runner
│   ├── ai/            # Prompt templates and GPT communication
│   ├── builder/       # File writer, bun installer, port allocator
│   ├── runner/        # Manages background app processes
│   └── registry.json  # Metadata of all apps
├── apps/              # All generated user apps
│   └── [app-name]/
├── runtime/           # Local bun binary, shared modules (optional)
├── gui/               # App0 frontend (React)
│   ├── AppList.tsx
│   ├── AppWindow.tsx
│   ├── CreateAppFlow.tsx
│   └── index.tsx
├── main.ts            # Electron entrypoint
├── package.json
└── README.md
```

### Key Components

**AI Agent (`core/ai/`)**
- Implements guided prompt flow to gather user requirements
- Converts requirements to specs and sends to OpenAI/GPT-4
- Uses strict generation prompts to receive full Next.js app code

**App Builder (`core/builder/`)**
- Creates new folders under `apps/APP_NAME/`
- Writes received files (package.json, pages/, api/, db/, etc)
- Manages bun installations and dependency management
- Handles port allocation for running apps

**App Runner (`core/runner/`)**
- Manages background app processes as subprocesses
- Tracks running ports and app states
- Handles app lifecycle (start, stop, restart)

**GUI (`gui/`)**
- React-based frontend for App0 interface
- Embeds running apps via iframe or webview
- Provides app creation flow, app list, and app viewer

**Registry (`core/registry.json`)**
- Stores metadata for all user apps
- Tracks app states, ports, and configurations

## Development Commands

Since this is a new project, these commands will need to be established during development:

**Electron Development:**
- `npm run dev` or `bun run dev` - Start Electron app in development mode
- `npm run build` or `bun run build` - Build the Electron application
- `npm run package` or `bun run package` - Package the app for distribution

**Generated Applets:**
- Generated apps use `bun dev` for development
- Generated apps use `bun start` for production
- All applet dependency management handled via bundled bun runtime

## Technical Constraints

- All generated applets are Next.js apps with SQLite and Tailwind
- App0 embeds a Bun runtime - users never install Node/npm
- Everything runs locally unless user chooses cloud upload
- No CLI commands or terminal usage exposed to end users
- All file operations, subprocesses, and AI calls handled internally
- Focus on simplicity and modularity

## Generated App Structure

Each generated applet follows this structure:
- Next.js framework
- SQLite database (if needed)
- Tailwind CSS for styling
- Self-contained in `apps/[app-name]/` directory
- Runs on random localhost port allocated by App0

## AI Integration

- Uses OpenAI GPT-4 for code generation
- Prompt templates stored in `core/ai/`
- Strict generation prompts ensure consistent Next.js output
- User requirements converted to specs before AI generation

## Sandboxing

- Each generated app is sandboxed
- Apps cannot access other apps' files
- Process isolation via separate bun dev instances
- Port isolation via random port allocation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Nuzair46) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
