## sentinelkioskjs

> Enterprise Electron kiosk platform that wraps web apps and bridges them to physical hardware (cash acceptors, coin validators, printers, NFC readers, barcode scanners). Real-time cloud reporting via AWS IoT Core. Full spec in `docs/KIOSK_PROJECT_PLAN.md`.

# KioskOS — Claude Code Project Guide

## Project Overview

Enterprise Electron kiosk platform that wraps web apps and bridges them to physical hardware (cash acceptors, coin validators, printers, NFC readers, barcode scanners). Real-time cloud reporting via AWS IoT Core. Full spec in `docs/KIOSK_PROJECT_PLAN.md`.

## Tech Stack

- **Runtime:** Electron 33+, Node.js 20 LTS
- **Language:** TypeScript 5.x (strict mode, entire codebase)
- **Bundler:** electron-vite
- **UI:** Vue 3 Composition API (admin screens only)
- **State:** Pinia
- **Local DB:** better-sqlite3 (WAL mode)
- **Package Manager:** pnpm 9+
- **Monorepo:** Turborepo with pnpm workspaces
- **Testing:** Vitest + @testing-library/vue + Playwright (E2E)
- **Linting:** ESLint flat config + Prettier
- **Logging:** winston (structured JSON)

## Commands

```bash
pnpm install                # Install all dependencies
pnpm run dev                # Start electron-vite dev mode with HMR
pnpm run dev:mock           # Dev mode with mock hardware adapters
pnpm run build              # Production build
pnpm run test               # Unit + integration tests (Vitest)
pnpm run test:watch         # Watch mode
pnpm run test:coverage      # Coverage report (gate: 80%)
pnpm run test:e2e           # Playwright E2E tests
pnpm run typecheck          # tsc --noEmit across all packages
pnpm run lint               # ESLint + Prettier check
```

## Repository Structure

```
kioskos/
├── apps/
│   ├── electron/            # The Electron application
│   │   └── src/
│   │       ├── main/        # Main process (ipc/, hardware/, db/, telemetry/, updater/, security/, admin/)
│   │       ├── preload/     # contextBridge exposing kioskAPI
│   │       ├── renderer/    # admin/ (Vue 3) + webview/ (wrapped web app)
│   │       └── shared/      # Shared TS interfaces and constants
│   └── cloud/               # AWS CDK infrastructure
├── packages/
│   └── shared-types/        # Shared types between electron + cloud
├── scripts/                 # Provisioning, build, seed scripts
└── docs/                    # Project plan, guides, runbooks
```

## Code Conventions

### Types first

Define interfaces in `shared/types/` before writing implementation. Every IPC channel must be typed in the `KioskAPI` interface before implementing handler and preload bridge.

### One adapter = one file

Each hardware adapter lives in its own file, named after the model (e.g., `CustomVKP80Adapter.ts`). All adapters extend the category abstract base class.

### Dependency injection

Pass dependencies (db, logger, config) via constructor. Never import singletons.

### Typed errors

Use `KioskError` base class with error codes. Hardware errors extend `HardwareError`.

### Testing

Write tests alongside implementation using AAA pattern (Arrange, Act, Assert). No module is complete without unit tests. Mock serial/USB ports for hardware adapter tests.

### Logging

Every hardware event, IPC call, sync operation, and state change gets a structured JSON log entry via winston.

### Environment guards

Dev-only features (mock hardware, dev tools, verbose logging) must be behind `IS_DEV` checks.

### Config

Loaded once at startup from a JSON file, merged with `local_config` DB overrides.

## Architecture Decisions

- **better-sqlite3** over Prisma — synchronous API, WAL mode, no ORM overhead
- **MQTT (IoT Core)** over WebSockets — purpose-built for IoT, mutual TLS
- **electron-vite** over Webpack — 10x faster HMR, native ESM
- **Adapter pattern** for hardware — new hardware = new file, zero core changes
- **BrowserView** over `<webview>` — more stable, not semi-deprecated
- **WireGuard** over OpenVPN — faster, lower overhead, simpler config

## Implementation Phases

Work through phases in order. Each phase should result in a working (if incomplete) application. See `docs/KIOSK_PROJECT_PLAN.md` Section 9 for full checklists.

1. Foundation — monorepo, electron-vite, IPC, BrowserView, SQLite, repos
2. Hardware Abstraction — adapter pattern, 5 categories, mock adapters
3. Cloud Infrastructure — CDK stacks, MQTT, telemetry, sync engine
4. Remote Management — IoT Jobs, auto-updater, rollback
5. Admin Panel — PIN/NFC auth, Vue admin pages
6. Alerting & Reporting — Slack, SES, CloudWatch alarms
7. Security & Hardening — VPN, CSP, code signing
8. Documentation & Polish — docs, E2E tests, perf profiling

## Style Preferences

- Keep code minimal — no premature abstractions or unnecessary helpers
- Prefer explicit over clever
- No comments unless the logic is non-obvious
- No emojis in code or commit messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rudedoc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
