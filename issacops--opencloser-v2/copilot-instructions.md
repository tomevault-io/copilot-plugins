## opencloser-v2

> OpenCloser is a Tauri 2.0 desktop AI sales platform built with React 19, TypeScript, Tailwind CSS, Rust, and SQLite. It provides an entire AI sales team: strategist, lead researcher, voice caller (SDR), coach, and manager — all running locally.

# OpenCloser — Agent Guide

## Project Overview

OpenCloser is a Tauri 2.0 desktop AI sales platform built with React 19, TypeScript, Tailwind CSS, Rust, and SQLite. It provides an entire AI sales team: strategist, lead researcher, voice caller (SDR), coach, and manager — all running locally.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | React 19, TypeScript 5.8, Tailwind CSS 3, Vite 6 |
| Desktop | Tauri 2.10 (Rust backend) |
| Database | SQLite (rusqlite) |
| AI | Google Gemini 2.5 Flash |
| Voice | Web Audio API + AudioWorklet |
| State | Zustand 5 |
| Tests | Vitest + React Testing Library |
| Icons | Lucide React |

## Project Structure

```
opencloser/
├── src/                          # Frontend source
│   ├── App.tsx                   # Root component
│   ├── main.tsx                  # Entry point
│   ├── constants.ts              # Storage keys, routes, nav config
│   ├── types.ts                  # Lead, ICP, Campaign, CallSession
│   ├── types/persona.ts          # AIPersona, voice presets
│   ├── index.css                 # Global styles, Tailwind, animations
│   ├── components/               # Shared components
│   │   └── AppShell.tsx          # Header + sidebar layout
│   ├── stores/                   # Zustand state management
│   │   ├── lead.store.ts         # Leads CRUD, search, filter
│   │   ├── call.store.ts         # Call state, active call, debrief
│   │   ├── onboarding.store.ts   # ICP interview flow
│   │   ├── persona.store.ts      # AI persona configuration
│   │   ├── navigation.store.ts   # App routing state
│   │   └── toast.store.ts        # Toast notifications
│   ├── services/                 # Tauri invoke wrappers
│   │   ├── lead.service.ts       # getLeads, updateLeadStatus, etc.
│   │   ├── onboarding.service.ts # processOnboardingChat
│   │   └── ai.service.ts         # Lead scraping, call analysis
│   ├── ui/components/            # Toast, future shared UI
│   ├── test/                     # Test files
│   │   ├── setup.ts              # Vitest setup
│   │   ├── emotion-engine.test.ts
│   │   ├── objection-engine.test.ts
│   │   └── providers.test.ts
│   └── features/
│       ├── crm/components/       # KanbanBoard, Dashboard, LeadDetail, Settings, PersonaBuilder, CallLogs
│       ├── voice/components/     # WarRoom, PostCallDebrief, ObjectionTrainer, VoiceVisualizer
│       ├── voice/lib/            # caller-engine, emotion-engine, objection-engine, providers
│       ├── hunter/components/    # LeadHunter
│       └── onboarding/components/ # Onboarding, ICPDisplay, AudioSetupWizard
├── src-tauri/                    # Rust backend
│   └── src/
│       ├── lib.rs                # Tauri app builder + invoke handlers
│       ├── main.rs               # Entry point
│       ├── ai/gemini.rs          # Gemini API + all mock/fallback logic
│       ├── db/schema.rs          # Schema init + migrations + seeds
│       └── db/commands.rs        # CRUD Tauri commands
├── public/
│   └── audio-processor.worklet.js # AudioWorklet PCM capture
├── vitest.config.ts              # Test configuration
├── tsconfig.json                 # TypeScript strict mode
├── tailwind.config.js
├── vite.config.ts
└── package.json
```

## Scripts

| Script | Purpose |
|--------|---------|
| `npm run dev` | Start Vite dev server (for web testing) |
| `npm run build` | Production build |
| `npm run lint` | TypeScript type check (strict mode) |
| `npm test` | Run all Vitest tests |
| `npm run test:watch` | Watch mode tests |
| `npm run test:coverage` | Test coverage report |
| `npm run tauri dev` | Start Tauri desktop app |
| `npm run tauri build` | Build desktop binaries |

## Conventions

### TypeScript
- **Strict mode enabled** — all code must pass `tsc --noEmit`
- Use explicit types for all function parameters and returns
- Avoid `any` — use `unknown` or proper interfaces
- Path alias: `@/` maps to project root

### Components
- Functional components (no classes)
- Props interfaces named `{ComponentName}Props`
- Export named functions, not defaults (except App.tsx)
- Use Tailwind utility classes, not inline styles (except dynamic values with `var()`)

### State
- Use Zustand stores in `src/stores/`
- Services in `src/services/` wrap all Tauri `invoke()` calls
- localStorage access goes through try/catch

### Demo / Fallback Mode
- When no `GEMINI_API_KEY` is set, all AI features fall back to realistic demo data
- Demo mode indicator shown in header
- All fallback data is in `src-tauri/src/ai/gemini.rs`

## Database

SQLite via rusqlite. 5 tables:
- `leads` — 12 columns including enriched contact fields
- `call_logs` — transcripts, sentiment, objection tracking
- `campaigns` — lead hunting campaigns
- `lead_notes` — freeform notes per lead
- `activities` — activity log

Migrations run automatically via `run_migrations()` in schema.rs. Seed data (10 leads + 3 call logs + 1 note) inserted on first run.

## CI/CD

- **CI** (push/PR to main): lint + test (Node 18/20/22) + build + cargo check
- **Release** (tag v*): builds for macOS/Linux/Windows via GitHub Actions, attaches binaries to draft release

## Contribution

1. Run `npm run lint` before committing
2. Run `npm test` before pushing
3. Add tests for new features
4. Update AGENTS.md if adding new directories/patterns

---
> Source: [issacops/opencloser-v2](https://github.com/issacops/opencloser-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
