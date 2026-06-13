## absen-kelas

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Absen Kelas is an offline desktop app (Tauri 2 + React 19) for Indonesian school operators to digitize student attendance from physical attendance books. All UI text is Bahasa Indonesia. No backend, no accounts — data lives in localStorage on the operator's machine.

## Commands

| Task | Command |
|---|---|
| Install deps | `npm install` |
| Dev server (browser) | `npm run dev` |
| Dev server (Tauri desktop shell) | `npm run tauri dev` |
| Run tests | `npm test` (runs `vitest run`) |
| Type-check + build frontend | `npm run build` (`tsc && vite build`) |
| Build desktop installer | `npm run tauri build` (requires Rust toolchain) |
| Release | Tag `vX.Y.Z` and push — GitHub Actions builds macOS/Windows installers and publishes to GitHub Releases |

## Architecture

**Monolithic UI**: All screens live in `src/App.tsx` (~1865 lines). Navigation is a `ViewKey` state variable switching between `"dashboard" | "attendance" | "students" | "classes" | "schedules" | "exports" | "settings"`. No router.

**Pure business logic in `src/lib/`**: All data transformations are pure functions that take `AppData` and return new `AppData` (immutable, spread-based updates). No React dependencies in this layer.

| File | Purpose |
|---|---|
| `types.ts` | All TypeScript interfaces (`AppData`, `Student`, `Class`, `AttendanceRecord`, etc.) |
| `attendance.ts` | Core CRUD, mark-all-present, export row builders |
| `storage.ts` | localStorage persistence (`absen-kelas:v1` key), backup/restore as JSON |
| `excel.ts` | Excel import/export via ExcelJS |
| `seed.ts` | Demo data for fresh installs |
| `status.ts` | Attendance status labels (Hadir, Izin, Sakit, Alpa, Tugas/Piket, Lainnya) |
| `dates.ts` | ISO date utilities, Indonesian date formatting |
| `ids.ts` | ID generator (prefix + timestamp + random) |

**Composite attendance key**: Records are keyed by `${date}:${classId}:${studentId}:${slotId}` in a flat `Record<>` map.

**UI components**: shadcn/ui primitives in `src/components/ui/` (radix-nova style, Lucide icons). Configured via `components.json`.

**Tauri backend** (`src-tauri/`): Minimal Rust layer — just `get_app_data_dir` and `open_app_data_dir` commands, plus plugins for opener, process, and updater.

## Path Alias

`@` maps to `./src/` (configured in both `vite.config.ts` and `tsconfig.json`).

## Testing

Tests use Vitest (configured in `vite.config.ts` with `environment: "node"`, `globals: true`). Test files are co-located: `src/lib/attendance.test.ts`, `src/lib/storage-info.test.ts`. Run a single test file with `npx vitest run src/lib/attendance.test.ts`.

## Product Principles (from CONTRIBUTING.md)

- Stay offline-first. No mandatory backend for core features.
- Target non-technical school operators.
- Use familiar Indonesian terminology.
- Keep scope tight: attendance, recap, import, export.

---
> Source: [ahmadghaisanfad2/absen-kelas](https://github.com/ahmadghaisanfad2/absen-kelas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-13 -->
