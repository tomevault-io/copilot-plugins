## nextjs-electron-migration-skill

> Use this skill when a user wants to convert or migrate a Next.js web app into a secure production-ready Electron desktop app. Apply it for Electron setup, secure preload and IPC contracts, desktop packaging, release hardening, and CI smoke tests, even when the user only says "make this a desktop app" without explicitly mentioning Electron architecture.


# Next.js to Electron Migration

Migrate Next.js apps to secure, maintainable Electron desktop apps using a repeatable production workflow.

## Use this skill

- Convert a Next.js web app into a desktop app.
- Add Electron main/preload process architecture.
- Define secure IPC patterns and renderer bridge APIs.
- Add packaging, signing-ready release flow, and CI smoke checks.
- Produce a migration plan and implementation checklist, not one-off hotfixes.

## Do not use this skill

- Minor web-only Next.js feature work without desktop requirements.
- Generic Electron bugfixes in an existing mature Electron codebase.
- Requests that only need PWA support instead of native desktop packaging.

## Default architecture

- Keep Next.js as the renderer app.
- Add `electron/main.(ts|js)` for lifecycle, windows, and IPC handlers.
- Add `electron/preload.(ts|js)` with a minimal `contextBridge` API.
- Keep all native capabilities behind explicit IPC contracts.
- Use an adapter in renderer (for example `src/lib/desktop/client.ts`) to isolate `window` access.

## Migration procedure

1. Audit app constraints: routing mode, SSR usage, API routes, environment variables, and static asset handling.
2. Add Electron runtime files and scripts for local desktop development.
3. Choose renderer serving model:
   - Development: load `next dev` URL.
   - Production default: run/build Next.js and load production server output.
4. Implement minimal IPC contracts first:
   - `desktop:get-app-version`
   - `desktop:shell:open-external`
5. Add desktop state persistence (for example, window bounds and user preferences).
6. Add packaging pipeline (for example, `electron-builder`) with platform targets.
7. Add smoke tests for app launch and critical IPC.
8. Add CI matrix for macOS, Windows, and Linux packaging checks.
9. Apply hardening gates (navigation policy, window policy, and permission boundaries).

## Security baseline (required)

- `contextIsolation: true`
- `nodeIntegration: false`
- `sandbox: true`
- Strict preload API whitelist; never expose raw Node.js objects to renderer.
- Validate all IPC inputs in main process before execution.
- Block unexpected navigation via `will-navigate`.
- Route external URLs through `setWindowOpenHandler` and trusted allowlists.
- Disable unneeded Electron features by default and opt in only when justified.

## High-value gotchas

- Do not put business logic in preload; preload is a narrow translation layer.
- Avoid broad `ipcRenderer.send` channels; use named, versioned contracts.
- Treat renderer data as untrusted input even if UI is internal.
- Ensure graceful shutdown of spawned Next.js processes in production mode.
- Keep development shortcuts (for example devtools auto-open) out of production builds.

## Validation loop (must run before completion)

1. Run lint/type/build checks for renderer and Electron code.
2. Start desktop app in dev mode and verify window load + basic navigation behavior.
3. Build production desktop package and validate cold start.
4. Verify at least one IPC roundtrip through preload with schema/input validation.
5. Verify external-link and navigation restrictions are enforced.
6. Re-run failed checks after fixes until all pass.

## Required output from the agent

When you apply this skill, return the result in this structure:

1. Scope and assumptions
2. Architecture decisions
3. File-level implementation plan
4. Security hardening actions
5. Build and packaging strategy
6. Test and CI strategy
7. Risks and mitigations
8. Definition of done

## Definition of done

- Desktop app works in development and production modes.
- Preload bridge is minimal, explicit, and validated.
- IPC contracts are implemented and input-validated in main process.
- Navigation and external-link restrictions are enforced.
- Packaging succeeds for required target platforms.
- CI smoke checks pass for launch and critical desktop behavior.

---
> Source: [Rana0001/nextjs-electron-migration-skill](https://github.com/Rana0001/nextjs-electron-migration-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
