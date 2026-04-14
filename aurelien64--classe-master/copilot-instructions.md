## classe-master

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Classe Master is an offline-first educational math PWA for French primary school students (CP-CM2). Built with SvelteKit 2 + Svelte 5 (runes), deployed as a static SPA with PWA support. All game logic runs client-side; Supabase is used for optional cloud sync.

## Commands

```bash
pnpm dev              # Start dev server
pnpm build            # Production build (static site)
pnpm preview          # Preview production build
pnpm check            # Type-check Svelte + TypeScript
pnpm lint             # ESLint + Prettier check
pnpm format           # Auto-format with Prettier
pnpm test             # Unit tests (Vitest, one-shot)
pnpm test:watch       # Unit tests in watch mode
pnpm test:e2e         # E2E tests (Playwright, requires build first)
```

Single test runs:

```bash
pnpm vitest run tests/unit/engine.test.ts          # Single unit test file
pnpm playwright test tests/e2e/onboarding.test.ts  # Single E2E test file
pnpm playwright test --project=chromium             # Single browser
pnpm playwright test --headed                       # With browser UI
```

E2E tests use `pnpm preview` (port 4173) as webserver. Playwright projects: chromium (Pixel 7 viewport 390x844) and firefox.

## Architecture

### Game Engine (`src/lib/engine/`)

Core math question generation pipeline, all pure TypeScript with no framework dependencies:

- **types.ts** — Core types: `Grade`, `Topic`, `QuestionType`, `TemplateConfig`, `Question`, `SessionState`
- **templates.ts** — Curriculum-aligned template configs defining operand ranges, constraints, and distractor strategies per grade/topic/sub-level (~1200 lines of data)
- **generator.ts** — Question generation from templates (with deduplication via fingerprinting, 10 retry attempts)
- **distractors.ts** — Wrong answer generation using pedagogically-relevant strategies (off_by_one, wrong_operation, forget_carry, etc.)
- **hints.ts** — 3-tier hint system (visual → decomposition → near-answer)
- **progression.ts** — Adaptive difficulty using rolling window of last 10 answers per topic (80% accuracy over 8+ questions to advance, drops level after 3 failed advancement attempts)
- **scoring.ts** — Score formula combining base points, speed bonus, difficulty multiplier, combo streak, and hint penalty

### Stores (`src/lib/stores/`)

Svelte 5 runes-based state management using `$state` and `$derived`:

- **player.svelte.ts** — Player profile, auth session (IndexedDB via idb-keyval + localStorage session token), XP/gems
- **game.svelte.ts** — Active game session state, question flow, timer (5min + 15s grace), answer validation

### Routes (`src/routes/`)

- `(auth)/join/` — Multi-step onboarding: mode selection → grade → avatar → username → optional PIN
- `(app)/` — Authenticated routes (menu, play, progress, settings) with layout guard checking localStorage session
- `+page.svelte` — Landing page

### i18n (`src/lib/i18n/`)

Uses svelte-i18n with JSON translation files. Default locale: French. Supports fr/en.

### Key Conventions

- **Svelte 5 runes only** — Use `$state`, `$derived`, `$props`, `{@render children()}`. No legacy `$:` reactive statements.
- **Tailwind CSS 4** — Utility-first styling, design tokens defined in `src/app.css` (CSS custom properties for colors, spacing, etc.)
- **Offline-first** — IndexedDB for data persistence, Service Worker caching via Workbox, sync queue for eventual Supabase upload
- **Privacy-first** — No personal data collection, anonymous UUIDs, profanity filter on usernames
- **Mobile-first** — Touch targets min 48px, portrait orientation, 390px reference viewport

### Formatting

- Tabs for indentation, single quotes, no trailing commas, 100 char print width
- Prettier with prettier-plugin-svelte

## Curriculum Structure

Grades: cp, ce1, ce2, cm1, cm2. Topics: addition, subtraction, multiplication, division, counting, ordering. Each topic has sub-levels 1-10 with progressively harder operand ranges defined in templates.ts. Curriculum specs live in `specs/curriculum-mapping-*.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aurelien64)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aurelien64)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
