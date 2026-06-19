## claude-commander

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Commander is a browser-based terminal multiplexer for managing multiple Claude Code instances with real-time hook event observability. It uses Next.js with Server-Sent Events (SSE) for real-time terminal output and hook event streaming.

## Essential Commands

### Development
- `pnpm dev` - Start Next.js development server with Turbopack
- `pnpm ai` - Run type checking and linting (use this instead of separate commands)
- `pnpm build` - Build for production
- `pnpm start` - Start production server

### Code Quality
- Type checking only: `pnpm ai:typecheck`
- Linting only: `pnpm ai:lint`

## Architecture

### Key Technologies
- **Next.js 15** with App Router
- **TypeScript** with strict mode
- **xterm.js** for terminal emulation
- **Server-Sent Events (SSE)** for real-time streaming
- **Tailwind CSS** with shadcn/ui components

### Core Components

1. **Process Management** (`src/lib/claude-process-manager.ts`):
   - Manages child processes for Claude Code instances
   - Currently spawns bash for testing (designed for Claude Code)
   - Tracks processes by instance ID

2. **Terminal Integration** (`src/components/ClaudeTerminal.tsx`):
   - xterm.js terminal with image pasting support (iTerm2 format)
   - Bidirectional communication with backend

3. **Real-time Communication**:
   - SSE endpoint: `/api/terminal/[instanceId]/stream` - Terminal output
   - HTTP POST: `/api/terminal/[instanceId]/input` - User input
   - Hook events: `/api/hooks` - Claude Code hook forwarding

4. **Hook System**:
   - External script (`scripts/hook-handler.ts`) receives Claude Code hooks
   - Events streamed to UI via SSE for real-time display

### Data Flow
1. User input → xterm.js → POST to server → Process stdin
2. Process output → SSE stream → xterm.js display
3. Claude hooks → hook-handler → API → SSE → UI display

## Important Notes

- The app currently uses **bash** instead of Claude Code for testing
- To integrate real Claude Code: Update spawn command in `claude-process-manager.ts`
- Image pasting converts to iTerm2 escape sequences automatically
- No test suite exists yet - verify changes manually in development

---
> Source: [brennancheung/claude-commander](https://github.com/brennancheung/claude-commander) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
