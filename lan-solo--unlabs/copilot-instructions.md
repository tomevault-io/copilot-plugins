## unlabs

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `pnpm dev` — Start Next.js dev server (port 3000)
- `pnpm build` — Production build
- `pnpm start` — Run the production build
- `pnpm lint` / `pnpm lint:fix` — ESLint (`eslint-config-next` + `eslint-config-prettier`)
- `pnpm typecheck` — `tsc --noEmit`
- `pnpm format` / `pnpm format:check` — Prettier
- `pnpm test` / `pnpm test:watch` — Vitest unit tests
- `pnpm test:e2e` — Playwright smoke tests (requires `pnpm exec playwright install --with-deps chromium` once)
- `pnpm check` — lint + typecheck + format:check + test + build (the same gates CI runs)
- `pnpm db:start` / `db:stop` / `db:reset` / `db:new <name>` / `db:diff` / `db:types` — Supabase CLI wrappers

See `docs/PIPELINE.md` for the full pipeline breakdown. Lefthook runs format/lint/typecheck on every commit and `pnpm test` on every push. GitHub Actions re-runs the same commands in CI.

Supabase local stack lives under `supabase/` (`config.toml`, `migrations/`). Use the Supabase CLI via the `db:*` scripts for migrations; never edit schema directly.

## Project Standards (non-negotiable)

### Language & Runtime

- TypeScript strict mode (already enabled in `tsconfig.json`)
- **PNPM only** — never `npm` or `yarn`
- Node.js 20+ / ES2022+
- Next.js 16 (App Router) + React 19

### Code

- Follow existing patterns before introducing new ones
- No `any` — use `unknown` and narrow with type guards
- Prefer named exports
- Use the `@/*` path alias for absolute imports (the only alias defined in `tsconfig.json`)

### Database

- Never modify the schema directly — always use migrations under `supabase/migrations/`
- Never run destructive queries (`DROP`, `TRUNCATE`) without explicit user confirmation

### Git

- Never force-push `main` or `develop`
- Never commit secrets or `.env` files
- Run lint and type-check before committing

### Architecture

- Keep business logic out of API route handlers — use service layers (`lib/api/*`, `lib/unos/*`)
- Prefer composition over inheritance

## High-Level Architecture

UnstableLabs is a Next.js App Router game that simulates a Linux-like operating system (`_unOS`) with a terminal, a hardware panel, and ~37 in-game devices. The non-obvious cross-cutting structure:

### Routes

- `app/(auth)/`, `app/auth/` — authentication
- `app/(game)/terminal/` — terminal UI (`terminal-frame.tsx`, `terminal-power-wrapper.tsx`, `actions/` for server actions)
- `app/(game)/panel/` — hardware panel UI (`panel-client.tsx`)
- `app/api/` — API routes; keep handlers thin and delegate to `lib/`

### \_unOS Kernel — `lib/unos/kernel/`

A simulated kernel with subsystems: `dmesg`, `process`, `memory`, `scheduler`, `syscall`, `ipc`, `modules`, `procfs`. Surrounding modules in `lib/unos/`: `filesystem`, `users`, `network`, `packages`, `containers`, `cron`, `journal`, `shell`, `devices`, `init`.

- The kernel is instantiated in `components/terminal/Terminal.tsx` via `kernelRef` and exposed through a `KernelActions` interface.
- `procfs` hooks into the virtual filesystem via `setProcFS()` / `setProcFSListDir()` to render dynamic `/unproc` content.
- Kernel state persists to `localStorage` through `PanelSaveData.kernel` (see `lib/panel/panelState.ts`).
- `init.ts` accepts an optional `Kernel` so processes get real PIDs from the process table.

### Terminal Subsystem — `lib/terminal/`

- `commands.ts` is **~18,500+ lines**. New commands must be registered in **two** places: (1) the command definition itself, and (2) the `commands[]` array at the bottom of the file. Missing the array registration is the most common bug.
- `types.ts` defines `DataFetchers` (~line 1340) — the contract every command uses to read game state and dispatch actions.
- `unapp/` (`appShell.ts`, `deviceApps.ts`, `moduleRenderers.ts`) implements in-terminal "apps" that render device modules.
- `hooks/useTerminal.ts` builds the `dataFetchers` object and wraps every command execution with `kernelActions.execCommand()` / `finishCommand()` so the kernel sees real processes.

### Device System — `contexts/` + `devices/` + `lib/firmware/`

Each in-game device has the same fan-out:

1. **Manager context** — `contexts/[XXX]Manager.tsx` (e.g. `VNTManager.tsx`, `CPUManager.tsx`). React provider holding state, firmware metadata, power specs, and action callbacks.
2. **Ref pattern** — `components/terminal/Terminal.tsx` collects every manager's ref and assembles an actions object.
3. **DataFetchers wiring** — `hooks/useTerminal.ts` wires those actions into `dataFetchers`.
4. **Command access** — terminal commands reach the device through `ctx.data.<actions>` (kernel actions live at `ctx.data.kernelActions`).
5. **UI module** — `components/panel/modules/` renders the device on the hardware panel.
6. **Docs/firmware metadata** — `devices/tier-{1,2,3}/<DEVICE>/` contains `DEVICE-ID.md` and `firmware.json`. `lib/firmware/registry.ts` and `lib/firmware/types.ts` model firmware at runtime; `contexts/FirmwareManager.tsx` is the runtime owner.

When adding device functionality, follow this exact pattern end-to-end — skipping the wiring layer is the second most common bug. See `devices/README.md` for the full device catalog and `devices/FIRMWARE-API.md` / `FIRMWARE-SPEC.md` for firmware contracts.

### Supabase / Data Layer

- Client setup in `lib/supabase/`, schema types in `types/database.ts` and `types/devices.ts`.
- All device DB operations go through `lib/api/devices.ts` (queries, runtime state, combinations, tweaks). System preferences split into `sysprefs.ts` (client) and `sysprefs-server.ts` (server actions) — server actions are the canonical loader for preference data.
- Recent perf work has consolidated sequential queries into RPCs/embeds/upserts; keep that pattern when adding new data flows.

### Panel Save Game

`lib/panel/panelState.ts` is the single source of truth for serialized game state (including kernel snapshot). Read it before designing any new persistence.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LAN-SOLO) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
