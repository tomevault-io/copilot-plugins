## extudio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Work Style

- **Implement first, ask later.** Do not brainstorm, present multiple options, or ask clarifying questions unless genuinely ambiguous. Default to building the thing.
- When making UI changes across the app, audit ALL views and components for consistency — don't stop at the obvious ones. Check sidebar states (collapsed/expanded), dock areas, and secondary views.

## Design System

- This project uses a **strictly monochrome black/white design system**. Never use colored accents, green dots, or brand colors unless explicitly requested.
- All icons must use `lucide-react`. Never use inline SVGs or colored icon variants.
- The only exception to monochrome is category colors (education=green, discovery=cyan, data=yellow, engagement=purple) which are used for tweet card accents only.

## File Conventions

- Use **kebab-case** for all file names (e.g., `compose-wizard.tsx`, not `ComposeWizard.tsx`).
- Use **TypeScript** for all code files.
- Use **Tailwind CSS** for styling — no plain CSS files except `globals.css`.

## Commands

- **Dev server:** `bun run dev` (Vite HMR)
- **Build:** `bun run build`
- **Lint:** `bun run lint`
- **Format:** `bun run format` (prettier)
- **Type check:** `npx tsc --noEmit`
- **Test:** `bun run test` (vitest) / `bun run test:watch`
- **Preview production build:** `bun run preview`
- **Install deps:** `bun install`
- **Storybook:** `bun run storybook` (serves on :6006) / `bun run build-storybook` → `storybook-static/`
- **Format check:** `bun run format:check`
- **Video preview:** `bun run video:preview` (Remotion)
- **Video render:** `bun run video:render`

## Architecture

React 19 + Vite 8 + TypeScript single-page app. Bun as the package manager. `@dnd-kit` for drag-and-drop. Zod for validation. Remotion + Three.js for video generation.

The app is **Extudio** — an open-source X/Twitter content studio (MIT License). Users configure it for their brand via `brand.json`. It has a 2-panel layout: sidebar nav and main workspace.

### Brand Configuration

All brand-specific data lives in `brand.json` at the project root. The typed loader at `src/config/brand.ts` imports and re-exports everything. **Never hardcode brand names, URLs, or domain-specific strings in source code** — always reference the brand config.

Key exports from `@/config/brand`:

- `appConfig` — app name, storage keys, default handle/display name
- `brandInfo` — name, URL, tagline, positioning, audience
- `glossary`, `contentPillars`, `tweetCategories`, `toneGuide`, `contentPatterns`, `seoKeywords`
- `audienceSegments`, `brandDifferentiators`, `brandMessages`, `categoryPlaybooks`, `brandKit`, `topicAngles`
- `bankTweets` — pre-written tweet bank
- `pillarToCategoryMap`, `categoryToPillarsMap` — category/pillar mappings

### Path aliases

`@/*` maps to `./src/*` (configured in both `tsconfig.json` and `vite.config.ts`).

### Views (via sidebar navigation, defined in `App.tsx`)

- **ComposeView** — Tweet/thread editor with AI prompt generation. Uses a wizard flow (`compose-wizard.tsx` → `compose-choice.tsx`). Stays mounted via `display: contents` so editor state persists across navigation.
- **QueueView** — 4-column Kanban board (draft → ready → scheduled → posted) with drag-and-drop
- **CalendarView** — Weekly schedule grid with slot management
- **PreviewView** — Tweet preview with X/Twitter-faithful rendering (`x-tweet-preview.tsx`)
- **ArticleView** — 3-tab article viewer (XArticlePanel, ThreadPanel, MarkdownPanel)
- **BrandView** — Brand context dashboard: glossary, tone guide, content pillars, SEO keywords, stats, patterns (click-to-copy)
- **OverviewView** — Dashboard/home view
- **BankView** — Pre-written tweet bank browser
- **ChatView** — AI chat interface
- **ReplyView** — Reply generation
- **ImagineView** — AI image studio
- **ProfileView** — User profile management
- **SettingsView** — App settings (schedule slots, AI mode, data export/import/clear)

`ViewId` type in `types.ts` defines the valid view identifiers: `compose | queue | calendar | article | brand | preview | overview | settings | reply | bank | profile | imagine | chat`.

### Data layer (`src/data/`)

- **store.tsx** — `useReducer` + React Context (`StudioProvider` / `useStudio`). Persists to localStorage using key from `appConfig.storageKey`.
- **contentEngine.ts** — Classifies content requests by type (tweet, thread, rewrite, variations, etc.), builds context-aware prompts with gap analysis

### Services (`src/services/`)

- **claude.ts** — AI content generation client. Calls `POST /api/generate` and streams results via SSE. Supports multiple providers in CLI mode (claude-code, openai-codex, open-code) and API mode (Anthropic, OpenAI, Gemini).
- **image.ts** — AI image generation service
- **x.ts** — X/Twitter API integration

### Vite plugins (`src/server/`)

Five custom Vite dev-server middleware plugins (dev-only — not bundled for production):

- **claude.ts** — Proxies AI generation requests to CLI tools or API providers
- **brand.ts** — Brand configuration handling
- **x.ts** — X/Twitter API proxy (OAuth 2.0)
- **image.ts** — Image generation proxy (OpenAI / Gemini)
- **persistence.ts** — File-backed state persistence (`/api/state`)

### Hooks (`src/hooks/`)

- `useTheme` — Dark/light theme toggle
- `use-streaming-generation` — SSE streaming for AI generation
- `use-brand` — Brand config access
- `use-account` / `use-x-connection` — X account state
- `use-config-status` — App configuration status
- `use-dialog-focus` / `use-escape-key` — UI interaction utilities

### Shared types (`src/types.ts`)

All shared interfaces and types: `Tweet`, `Topic`, `ScheduleSlot`, `StudioState`, `StudioAction`, `NavigateData`, `ViewId`, `ContentType`, `ContentRequest`, `EngineResult`, `BrandConfig`, `AppConfig`, etc.

### Component structure (`src/components/`)

- **`layout/`** — App shell: `sidebar.tsx`, `app-background.tsx`
- **`tweet/`** — Tweet rendering: `tweet-card.tsx`, `tweet-editor-card.tsx`, `x-tweet-preview.tsx`
- **`shared/`** — Cross-view components: `brand-wizard.tsx`, `connect-x-modal.tsx`, `settings-modal.tsx`, `toolbar.tsx`, `toast.tsx` (imperative via `showToast()`), `error-boundary.tsx`, `hydration-status.tsx`, `schedule-modal.tsx`, `post-confirm-modal.tsx`
- **`ui/`** — shadcn/ui primitives and app-level atoms (Button, IconBox, Badge, Select, FloatingBar, EmptyState, BubbleTabs, etc.)
- **`views/`** — One subdirectory per view (compose, queue, calendar, etc.)

## Styling

Tailwind CSS v4 with `@tailwindcss/postcss`. shadcn/ui components (`base-mira` style). All CSS custom properties defined in `src/globals.css` — includes a full design system with surface/text/border semantic tokens and category colors (education=green, discovery=cyan, data=yellow, engagement=purple). Light/dark theme support via `.dark` class toggle (`src/hooks/useTheme.ts`). Font: Poppins.

### Video (`src/video/`)

Remotion-based video generation for launch/promo videos. Separate from the main app — uses `@remotion/cli`, `@react-three/fiber`, and Three.js.

## ESLint

Configured for both `.js/.jsx` and `.ts/.tsx` files via `eslint.config.ts` (flat config). Key rules:

- `no-unused-vars` / `@typescript-eslint/no-unused-vars`: ignores variables starting with uppercase or underscore (`varsIgnorePattern: '^[A-Z_]'`, `argsIgnorePattern: '^_'`)
- `@typescript-eslint/no-unused-expressions`: warn
- `react-refresh/only-export-components`: warn

---
> Source: [owklama/extudio](https://github.com/owklama/extudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
