## musiqasik

> <!-- SYNC NOTE: AGENTS.md and CLAUDE.md must be kept identical. Edit AGENTS.md first, then copy to CLAUDE.md -->

<!-- SYNC NOTE: AGENTS.md and CLAUDE.md must be kept identical. Edit AGENTS.md first, then copy to CLAUDE.md -->

# MusiqasiQ - Agent Documentation

## What is MusiqasiQ?

A React-based web application that visualizes artist similarity relationships through interactive force-directed graphs. Integrates with Last.fm API and provides an engaging interface for exploring music artist connections.

## Tech Stack

- **Frontend**: React 18 + TypeScript + Vite + Tailwind CSS + shadcn/ui + D3.js
- **Services**: Effect library for typed, composable service architecture
- **API Caching**: Cloudflare Workers (optional local development)
- **State**: React hooks for local state, Effect runtime for service operations
- **Routing**: React Router DOM

## Project Structure

```
musiqasik/
├── src/
│   ├── components/         # React components
│   │   ├── ForceGraph/     # D3.js graph visualization (refactored into hooks)
│   │   └── ui/             # shadcn/ui components
│   ├── hooks/              # React hooks (useLastFm, useSimilarArtists, etc.)
│   ├── services/           # Effect services (LastFm, Database, Graph, Config)
│   ├── integrations/       # External service clients
│   ├── lib/                # Utilities and error types
│   ├── pages/              # Route pages
│   └── types/              # TypeScript type definitions
├── e2e/                    # Playwright E2E tests
└── public/                 # Static assets
```

## Getting Started

1. **Setup**: Copy `.env.example` to `.env` and add your Last.fm API key
2. **Development**: `bun run dev` starts server on port 8080
3. **Building**: `bun run build` for production
4. **Testing**: `bun run test` for unit tests, `bun run test:e2e` for E2E tests
5. **Linting**: `bun run lint` runs Biome

## Architecture Overview

- **Data Flow**: Search → useLastFm hook → Effect runtime → LastFmService → Last.fm API → Graph
- **Graph Visualization**: D3.js force simulation with modular hooks (`ForceGraph/hooks/`)
- **Caching**: Optional Cloudflare Workers caching
- **State**: URL params for shareable graphs, React hooks for UI state
- **Service Layer**: Effect-based services with proper dependency injection

### Key Services (in `src/services/`)

- **ConfigService**: Environment configuration (API keys, DB URLs)
- **LastFmService**: Last.fm API integration (search, artist info, similar artists)
- **DatabaseService**: Cloudflare KV operations (artist caching)
- **GraphService**: BFS graph building algorithm

## How to Work on This Project

### Before Starting

- Read relevant documentation files based on your task
- Study existing code patterns in similar components/files
- Check `package.json` for available scripts

### Documentation Files (Progressive Disclosure)

Consult these files for detailed information:

- `agent_docs/development-workflow.md` - Setup, scripts, environment
- `agent_docs/architecture-patterns.md` - System design and data flow
- `agent_docs/code-conventions.md` - Patterns to follow (learn from existing code)
- `agent_docs/common-tasks.md` - Step-by-step guides for common operations
- `agent_docs/troubleshooting.md` - Debugging and common issues
- `agent_docs/effect.md` - Effect library patterns and API reference

### Key Principles

1. **Follow existing patterns**: Study similar components before creating new ones
2. **Use TypeScript**: Interfaces in `src/types/`, optional properties with `?`
3. **Use Effect for services**: Services use Effect library for composable, typed operations
4. **Style with Tailwind**: Use `cn()` utility for conditional classes (`src/lib/utils.ts`)
5. **Handle errors**: AppError types in `src/lib/errors.ts`, toast notifications for user feedback
6. **Write tests**: Unit tests with Vitest, E2E tests with Playwright

### Important Notes

- **TypeScript strict mode is disabled** (`tsconfig.app.json`)
- **Path aliases**: `@/` maps to `./src/` (`vite.config.ts`)
- **Environment variables**: Must be prefixed with `VITE_` for client-side access
- **Caching optional**: App works without Cloudflare Workers using Last.fm directly

## Testing

- `bun run test` - Unit tests (hooks, utilities, services)
- `bun run test:e2e` - E2E tests (Playwright)
- `bun run test:coverage` - Coverage report

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adrioui)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/adrioui)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
