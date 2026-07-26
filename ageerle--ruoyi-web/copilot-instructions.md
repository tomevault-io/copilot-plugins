## ruoyi-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **ruoyi-web**, a Vue 3 AI chat application built with TypeScript, Vite, and Element Plus. It provides a chat interface for AI interactions with support for sessions, models, and various authentication methods.

## Commands

```bash
# Development
pnpm dev          # Start dev server

# Build
pnpm build        # Type-check and build for production

# Linting
pnpm lint         # Run ESLint
pnpm lint:stylelint  # Run Stylelint
pnpm fix          # Auto-fix ESLint issues

# Preview production build
pnpm preview
```

## Architecture

### Tech Stack
- **Vue 3** with Composition API (`<script setup>`)
- **TypeScript** for type safety
- **Pinia** for state management with persistence (`pinia-plugin-persistedstate`)
- **Vue Router** for routing with navigation guards
- **Element Plus** for UI components
- **UnoCSS** for atomic CSS
- **hook-fetch** for HTTP requests with SSE support

### Directory Structure

```
src/
├── api/           # API modules (auth, chat, model, session) with types
├── assets/        # Static assets (SVG icons organized by category)
├── components/    # Reusable components (LoginDialog, ModelSelect, etc.)
├── config/        # App configuration constants
├── constants/     # Enum definitions
├── hooks/         # Custom Vue composables
├── layouts/       # Layout components (LayoutVertical, LayoutMobile)
├── pages/         # Page components (chat, error pages)
├── routers/       # Route definitions and guards
├── stores/        # Pinia stores (user, chat, session, model, design)
├── styles/        # Global SCSS styles and variables
├── utils/         # Utilities (request wrapper, markdown renderers)
└── main.ts        # App entry point
```

### Key Patterns

**API Layer** (`src/api/`):
- Each module has `index.ts` for API calls and `types.ts` for TypeScript interfaces
- Uses `hook-fetch` with JWT plugin for authentication
- Base URL configured via `VITE_API_URL` environment variable

**State Management** (`src/stores/`):
- Stores use Composition API style with `defineStore`
- User store persists token and userInfo
- Session store handles chat sessions with pagination
- Chat store manages messages and deep thinking state

**Routing** (`src/routers/`):
- Static routes defined in `modules/staticRouter.ts`
- Route guard checks token and handles auth redirect
- White list routes bypass auth check

**HTTP Requests** (`src/utils/request.ts`):
- Auto-injects `Bearer` token and `ClientID` headers
- Handles 401 (logout) and 403 (redirect) responses
- Supports SSE streaming via `sseTextDecoderPlugin`

### Environment Variables

Configure in `.env.development`:
- `VITE_API_URL` - Backend API base URL
- `VITE_CLIENT_ID` - Client identifier for auth
- `VITE_WEB_TITLE` - Page title

### Code Style

- ESLint with `@antfu/eslint-config`
- Vue blocks order: script → template → style
- Single quotes, semicolons, 2-space indent
- Commit messages follow conventional commits with commitlint

---
> Source: [ageerle/ruoyi-web](https://github.com/ageerle/ruoyi-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
