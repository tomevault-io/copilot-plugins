## ats-chd-tools

> **Name**: ATS CHD Tools

# Gemini Workspace Context

## Project Overview
**Name**: ATS CHD Tools
**Description**: Unified desktop platform to replace Excel workflows for the ATS CHD department. The application is a single-user, offline Windows desktop app built with Tauri.
**Current Focus**: Phase 2 - Testing Infrastructure, EPLAN Export, Heat/Load Calculation, and Quoting modules.

## Technology Stack
- **Framework**: React 19 + TypeScript 5.9
- **Build Tool**: Vite 7
- **Desktop/Native**: Tauri 2.9 (Rust)
- **Styling**: TailwindCSS 4, shadcn/ui
- **State Management**: Zustand 5
- **Form Handling**: react-hook-form + zod
- **Database**: SQLite (via @tauri-apps/plugin-sql)
- **Routing**: react-router-dom 7

## Project Structure
- `.prd/`: Product Requirements Documents and active sprint planning.
- `docs/`: Project documentation and architecture decisions.
- `openspec/`: Project specifications and change tracking.
- `src/`: Frontend source code (React components, pages, lib).
- `src-tauri/`: Backend source code (Rust, Tauri configuration).
- `src-tauri/migrations/`: Database migration files.
- `src/lib/db/client.ts`: Centralized database query logic.

## Key Modules
1. **BOM Translation**: Import, manage, and export Bill of Materials (Completed).
2. **Parts Library**: Library for managing and searching parts (Completed).
3. **Glenair Builder**: Configurator for Glenair Series 80 connectors (Completed).
4. **EPLAN Export**: BOM export in EPLAN XML format (Active).
5. **Heat/Load Calculation**: Thermal and load calculation workflows (Active).
6. **Quoting**: Pricing, cost estimation, and quote generation (Active).

## Development Commands
- `npm run dev`: Start frontend dev server.
- `npm run tauri:dev`: Start Tauri dev environment.
- `npm run tauri:build`: Build for production.
- `npm run db:reset`: Reset database.
- `npm run db:seed:bom`: Seed BOM test data.

## Deployment
- **Target**: Windows Desktop
- **Environment**: Offline, Single-user

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tybradle)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tybradle)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
