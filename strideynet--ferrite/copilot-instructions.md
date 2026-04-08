## ferrite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ferrite is an amateur radio suite of software written in Rust. The main application is a website for amateur radio logging with:

- **Backend**: Rust
- **Frontend**: TypeScript, React, Vite, Shadcn UI

## Architecture

### Backend (Rust)

The backend handles amateur radio logging operations, data persistence, and API endpoints.

### Frontend (TypeScript/React)

The frontend is built with:

- **React 19** for UI components
- **Vite 7** for build tooling and development server
- **Shadcn UI** for component library (New York style)
- **Tailwind CSS v4** for styling
- **TypeScript** with strict mode enabled

Path aliases are configured using `@/` which maps to `./src/`. For example:
- `@/components/ui/button` → `src/components/ui/button.tsx`
- `@/lib/utils` → `src/lib/utils.ts`

## Development Commands

### Frontend

```bash
cd web-log-frontend
pnpm install                      # Install dependencies
pnpm dev                          # Start development server
pnpm build                        # Build for production (runs TypeScript compiler + Vite build)
pnpm lint                         # Run ESLint
pnpm dlx shadcn@latest add <component>  # Add a new Shadcn UI component
```

### Backend

```bash
cd web-log-backend
cargo build           # Build the backend
cargo test            # Run tests
cargo run             # Run the backend server
```

## Project Structure

The project is organized with separate backend and frontend directories:

- `web-log-backend/` - Rust backend server (to be created)
- `web-log-frontend/` - React/TypeScript frontend
  - `src/components/ui/` - Shadcn UI components
  - `src/lib/utils.ts` - Utility functions (includes `cn()` for className merging)
  - `components.json` - Shadcn UI configuration

## Notes

- The repository uses pnpm for Node.js package management
- Path aliases use `@/` to reference the `src/` directory
- Shadcn UI components should be added using `pnpm dlx shadcn@latest add <component>`
- Ignore the `ferrite-contest-log/` directory - it's not part of the main project
- The project is in early development stages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/strideynet)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/strideynet)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
