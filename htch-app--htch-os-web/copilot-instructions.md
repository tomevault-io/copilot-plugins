## htch-os-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a TanStack Start application deployed on Cloudflare Workers. It's a full-stack React framework with server functions, streaming support, and type-safe API endpoints. The project uses Vite for building, Vitest for testing, and TanStack Router for file-based routing.

## Common Commands

### Development

- `pnpm dev` - Start development server (runs on port 3000)
- `pnpm build` - Build for production
- `pnpm preview` - Preview production build locally
- `pnpm serve` - Same as preview

### Testing & Code Quality

- `pnpm test` - Run Vitest tests
- `pnpm lint` - Format and lint code with Biome (auto-fixes issues)

### Deployment

- `pnpm run deploy` - Build and deploy to Cloudflare Workers
- `pnpm run cf-typegen` - Generate Cloudflare Worker types

### Pre-commit Hooks

Lefthook is configured to run `pnpm lint` before commits. This ensures code formatting and linting issues are caught before committing.

## Architecture

### Tech Stack

- **Frontend Framework**: React 19 with TanStack Router v1
- **Build Tool**: Vite with TanStack React Start plugin
- **Styling**: Tailwind CSS v4
- **Testing**: Vitest with React Testing Library
- **Linting**: Biome (with git integration)
- **Icons**: lucide-react
- **Deployment**: Cloudflare Workers (via Wrangler)

### Project Structure

```
src/
├── components/       # Reusable React components (e.g., Header.tsx)
├── routes/          # File-based routing with TanStack Router
│   ├── __root.tsx   # Root layout and document shell
│   ├── index.tsx    # Home page
│   └── demo/        # Demo routes (marked for deletion - these are examples)
├── data/            # Static data files
├── styles.css       # Global styles with Tailwind
├── router.tsx       # Router configuration
└── routeTree.gen.ts # Auto-generated route tree (DO NOT EDIT)
```

### Routing

TanStack Router uses **file-based routing** in `src/routes/`. Each file automatically becomes a route:

- `src/routes/index.tsx` → `/`
- `src/routes/about.tsx` → `/about`
- `src/routes/demo/start.server-funcs.tsx` → `/demo/start/server-funcs`

The `routeTree.gen.ts` file is auto-generated - never edit it manually. Routes are configured using `createFileRoute()` and `createRootRoute()` from `@tanstack/react-router`.

**Root Layout**: `src/routes/__root.tsx` defines the document shell and global layout. It wraps all routes with the `Header` component and devtools.

### Server Functions & API Routes

TanStack Start supports:

1. **Server Functions** - Functions marked with `"use server"` that run on the server but are callable from client components
2. **API Routes** - Files in `src/routes/` that export `GET`, `POST`, etc. handlers

Routes can optionally include a `loader()` function that runs before rendering to fetch data.

### Configuration Files

- **`vite.config.ts`** - Vite build config with TanStack Start, Cloudflare, and Tailwind plugins
- **`wrangler.jsonc`** - Cloudflare Worker configuration (SSR entry point, compatibility, observability)
- **`tsconfig.json`** - TypeScript config with strict mode, path aliases (`@/*` → `src/*`)
- **`biome.json`** - Linter and formatter config with git integration, tab indentation, double quotes
- **`lefthook.yml`** - Git hooks config that runs `pnpm lint` before commits

## Development Tips

### Type Safety

The project has strict TypeScript enabled. Use the `@/*` path alias for imports:

```tsx
import Header from "@/components/Header";
```

### Styling

- Use Tailwind CSS utility classes for styling
- Global styles go in `src/styles.css`
- Tailwind v4 is configured in the Vite pipeline

### Adding Routes

Create a new `.tsx` file in `src/routes/` with:

```tsx
import { createFileRoute } from "@tanstack/react-router";

export const Route = createFileRoute("/your-route")({
  component: YourComponent,
});

function YourComponent() {
  return <div>Your content</div>;
}
```

The route tree auto-generates - no manual registration needed.

### Testing

Tests should be colocated with or named after their corresponding components/utilities. Vitest discovers files matching `**/*.test.ts(x)` or `**/*.spec.ts(x)`.

## Deployment Notes

The app deploys to Cloudflare Workers with the main entry point configured in `wrangler.jsonc` as `@tanstack/react-start/server-entry`. This enables SSR (Server-Side Rendering) on Cloudflare's edge.

Node.js compatibility is enabled via `nodejs_compat` compatibility flag.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HTCH-app)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/HTCH-app)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
