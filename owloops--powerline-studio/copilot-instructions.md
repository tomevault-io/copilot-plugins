## powerline-studio

> Powerline Studio is a visual configuration editor for [claude-powerline](https://github.com/owloops/claude-powerline), a vim-style statusline for Claude Code. It renders a live terminal preview using the real claude-powerline rendering engine, lets users configure themes, styles, segments, and TUI layouts visually, then export the config JSON.

# CLAUDE.md

## Project Overview

Powerline Studio is a visual configuration editor for [claude-powerline](https://github.com/owloops/claude-powerline), a vim-style statusline for Claude Code. It renders a live terminal preview using the real claude-powerline rendering engine, lets users configure themes, styles, segments, and TUI layouts visually, then export the config JSON.

Single-page Vue app — one page at `/`, one layout, no backend required.

## Tech Stack

| Tool                                | Role                                                 |
| ----------------------------------- | ---------------------------------------------------- |
| Vue 3.6 + Composition API           | UI framework, `<script setup lang="ts">` exclusively |
| Vite 8 via VitePlus (`vp` CLI)      | Build toolchain                                      |
| Tailwind CSS 4                      | Styling (CSS-first config, `@theme {}` blocks)       |
| shadcn-vue + reka-ui                | UI components (owned copies in `src/components/ui/`) |
| Pinia 3                             | State management (4 stores)                          |
| `@owloops/claude-powerline/browser` | Rendering engine (local linked dependency)           |
| ansi_up                             | ANSI escape codes → HTML conversion                  |
| Shiki                               | JSON syntax highlighting in export panel             |
| Motion for Vue                      | Animations                                           |
| VueUse                              | Utility composables (auto-imported)                  |

## Commands

```bash
vp dev              # Dev server + HMR
vp build            # Production build
vp preview          # Preview production build
vp check            # Lint + format + type-check
vp check --fix      # Auto-fix
vp test             # Vitest
pnpm test:e2e       # Playwright E2E
```

## Architecture

### Rendering Pipeline

```
Config Store → SegmentRenderer/renderTuiPanel → ANSI string → ansi_up → HTML <pre>
```

The `useRenderer` composable watches all stores and reactively produces HTML output with clickable segment hitboxes overlaid on the terminal preview.

### File Organization

```
src/
├── pages/(home).vue              # Single page — the studio editor
├── layouts/default.vue           # Full-height layout wrapper
├── components/
│   ├── studio/                   # All studio-specific components
│   │   ├── panels/               # Config panels (Theme, Style, Segments, TUI, MockData, Export)
│   │   ├── segments/             # Per-segment config forms (13 segments)
│   │   ├── theme/                # Theme picker and custom editor
│   │   ├── tui/                  # TUI grid layout editor
│   │   └── mockdata/             # Mock data forms for preview
│   ├── ui/                       # shadcn-vue components (owned, modifiable)
│   └── Form*.vue                 # Shared form field wrappers (FormWerk + shadcn)
├── composables/
│   ├── useRenderer.ts            # Core rendering pipeline
│   ├── useDarkMode.ts            # Dark mode toggle
│   ├── useShikiHighlighter.ts    # Shiki syntax highlighting
│   └── useTuiValidation.ts       # TUI layout validation
├── stores/
│   ├── config.ts                 # PowerlineConfig — all config mutations
│   ├── editor.ts                 # UI state — selected segment, active panel
│   ├── mockData.ts               # Mock data for preview (presets + custom)
│   ├── preview.ts                # Terminal width, color mode, rendered output
│   └── utils.ts                  # deepMerge utility
├── types/
│   ├── studio.ts                 # Studio panel types
│   └── tui.ts                    # TUI layout types
├── assets/
│   ├── main.css                  # Tailwind theme, shadcn vars, Nerd Font face
│   └── fonts/                    # FiraCode Nerd Font (for powerline symbols)
└── main.ts                       # App entry (ViteSSG + Pinia)
```

### Key Dependency

`@owloops/claude-powerline` provides the browser-safe rendering engine (`/browser` entry point) with zero Node.js imports.

## Code Conventions

- `<script setup lang="ts">` — never Options API
- Tailwind CSS 4 exclusively — no inline styles
- `cn()` from `@/lib/utils` for conditional class merging
- Auto-imports: Vue APIs, VueUse, composables, stores — no manual imports needed
- Icons as `<IconLucide-{name} />` via unplugin-icons
- Tab indentation, no semicolons, single quotes

## Important Notes

- VitePlus manages Vite — don't install `vite`, `vitest`, `oxlint`, `oxfmt` directly
- Import from `vite-plus` (not `vite`) and `vite-plus/test` (not `vitest`)
- The terminal preview requires FiraCode Nerd Font (self-hosted woff2 in `src/assets/fonts/`)
- Config persists to `localStorage` via VueUse's `useStorage` in the config store

---
> Source: [Owloops/powerline-studio](https://github.com/Owloops/powerline-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
