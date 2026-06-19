## claude-code-cli-training

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## What this repo is

**claude-code-cli-training** — an interactive, bilingual (EN/ES) learning platform for Claude Code built on **Astro 5**. The primary UI is a real xterm.js terminal with a step-by-step sidebar. Everything is scripted and deterministic — no LLM backend.

Live at: https://claude-code-cli-training.vercel.app

The v1 Docusaurus prototype is archived under `legacy/`.

## Commands

```bash
npm install
npm run dev      # dev server at http://localhost:4321
npm run build    # static output to dist/
npm run preview  # preview the production build
```

## Architecture

### Stack

- **Astro 5** — static output, `output: 'static'`, Vercel adapter
- **React islands** (`client:only="react"`) — Terminal, Sidebar, StepsPanel are the hydrated components
- **xterm.js** (`@xterm/xterm`, `@xterm/addon-fit`, `@xterm/addon-web-links`) — real terminal emulator in the browser
- **Tailwind CSS** — `preflight: false` to avoid style collisions, `cc-*` VSCode color palette
- **Zustand** — progress store with localStorage persistence (`claude-code-cli-training:progress`)
- **Astro Content Collections** — MDX lesson content validated by Zod at build time

### Layout

4-column flex layout per lesson page:

```
┌──────────┬────────────┬──────────────┬─────────────┐
│  Sidebar │ StepsPanel │   Terminal   │Detail drawer│
│  (288px) │  (340px)   │   (flex-1)   │  (320px,    │
│  course  │  per-step  │  xterm.js +  │  hidden by  │
│   nav +  │   cards    │  input bar   │  default)   │
│  controls│            │              │             │
└──────────┴────────────┴──────────────┴─────────────┘
```

Detail drawer is toggled via `window.dispatchEvent(new CustomEvent('toggle-detail'))`.

### i18n

Native Astro i18n:
- `defaultLocale: 'en'` — English served at `/`, no prefix
- Spanish served at `/es/`
- `routing.prefixDefaultLocale: false`

### Directory layout

```
src/
├── components/
│   ├── Terminal/         # xterm.js island
│   │   ├── Terminal.tsx  # main component, mode state, input bar, macOS chrome
│   │   ├── ScriptedPty.ts# line editor, step evaluation, shell/claude dual input
│   │   ├── BootSequence.ts# shell (Last login) and claude (mascot+banner) variants
│   │   ├── ansi.ts       # ANSI color helpers
│   │   └── theme.ts      # xterm color theme
│   ├── Sidebar/          # course navigation, hints, show-solution, restart
│   └── StepsPanel/       # per-lesson numbered step cards with progress
├── content/
│   ├── config.ts         # Zod schema for Content Collections
│   └── lessons/
│       ├── en/<section>/<slug>.mdx   (18 files)
│       └── es/<section>/<slug>.mdx   (18 files)
├── lessons/              # scripted lesson definitions (TypeScript)
│   ├── index.ts          # LESSONS registry + SECTIONS metadata
│   ├── types.ts          # Step, Lesson, LessonMode types
│   ├── getting-started/  (5 files)
│   ├── cli-fundamentals/ (6 files)
│   └── core-features/    (8 files)
├── stores/
│   └── progress.ts       # Zustand store, persists to localStorage
├── layouts/
│   ├── BaseLayout.astro
│   └── LessonLayout.astro
├── pages/
│   ├── index.astro               # EN landing (renders installation lesson)
│   ├── [section]/[slug].astro    # EN lesson pages
│   └── es/
│       ├── index.astro           # ES landing
│       └── [section]/[slug].astro
└── styles/global.css             # Tailwind + xterm overrides
```

### Lesson step shape

```ts
export type LessonMode = 'shell' | 'claude';

type Step = {
  id: string;
  label?: string;           // displayed in StepsPanel card
  mode?: LessonMode;        // per-step override (falls back to lesson.defaultMode ?? 'shell')
  expect: RegExp | string | (RegExp | string)[];  // alias arrays supported
  response: string;         // ANSI pre-formatted string
  hint?: string;            // shown on mismatch
  solution?: string;        // auto-typed by "Show solution" button
  thinking?: string;        // label shown during thinking animation (claude mode only)
  delay?: number;
};

type Lesson = {
  id: string;
  steps: Step[];
  defaultMode?: LessonMode;
};
```

### Shell mode vs Claude mode

Each step has an effective mode: `step.mode ?? lesson.defaultMode ?? 'shell'`.

- **Shell mode**: xterm handles input via `term.onData`. Prompt is `projects [main] ⚡`. Output appears instantly. Arrow keys move cursor within typed text. Slash commands are always instant.
- **Claude mode**: React input bar at the bottom of the terminal handles input. Prompt is `❯` (orange). Prose responses use typewriter effect. Slash commands appear instantly.

When a step's mode differs from the previous step's mode, the terminal transitions:
- `shell → claude`: runs the Claude boot sequence (mascot + version + model)
- `claude → shell`: prints `(session ended)` and returns to shell prompt

A `ContextBadge` above the terminal window always shows the current mode.

### Authoring a new lesson

1. Create `src/lessons/<section>/<slug>.ts` exporting a `Lesson`.
2. Register in `src/lessons/index.ts` (import + add to `LESSONS`; bump `SECTIONS[section].total`).
3. Add MDX to `src/content/lessons/en/<section>/<slug>.mdx` and `src/content/lessons/es/<section>/<slug>.mdx`.
4. `npm run build` validates the wiring.

### Content Collection schema (Zod)

Frontmatter fields: `id` (string), `title` (string), `intro` (string), `section` (enum), `position` (number), `references` (optional array of `{label, url}`).

## Gotchas

- **`node_modules/` is gitignored** — always run `npm install` in a fresh clone.
- **JetBrains Mono is self-hosted** — fonts live in `public/fonts/` as `.woff2` (400/500/600/700). No Google CDN dependency.
- **xterm CSS** — `@xterm/xterm/css/xterm.css` must be imported inside `Terminal.tsx` (the island), not globally, or SSR will fail.
- **Deploy target is Vercel** — `astro.config.mjs` uses `@astrojs/vercel` adapter with `output: 'static'`.
- **`writePrompt()` must be called after `runBootSequence()`** — boot is async; chain `.then(() => pty.writePrompt())`.
- **Arrow keys in shell mode** — `onData` fires the full escape sequence (`\x1b[D` etc.) as one string. Handle before the char loop.
- **Typewriter is claude-mode only** — shell mode and slash commands always render instantly.
- **Legacy prototype** — `legacy/` contains the full Docusaurus v1 code. Use it as reference only.

---
> Source: [davila7/claude-code-cli-training](https://github.com/davila7/claude-code-cli-training) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
