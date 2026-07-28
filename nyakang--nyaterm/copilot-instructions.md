## nyaterm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development commands

### Root app
- `pnpm install` — install JS dependencies
- `pnpm dev` — run the Vite frontend only
- `pnpm tauri dev` — run the full desktop app in Tauri dev mode
- `pnpm build` — run `tsc` and build the frontend with Vite
- `pnpm tauri build` — build the production desktop bundle
- `pnpm lint` — run Biome checks for `src/**/*.ts` and `src/**/*.tsx`
- `pnpm format` — apply Biome formatting to `src/**/*.ts` and `src/**/*.tsx`
- `pnpm format:check` — check Biome formatting without writing changes
- `pnpm i18n:check` — check locale JSON formatting
- `pnpm i18n:fix` — rewrite locale JSON formatting
- `pnpm version-sync` — sync version numbers across app files
- `pnpm release` — version sync + frontend build + Tauri build

### Rust / Tauri backend
- `cargo fmt --manifest-path src-tauri/Cargo.toml` — format Rust code
- `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets` — lint Rust code
- `cargo test --manifest-path src-tauri/Cargo.toml` — run backend Rust tests
- `cargo test --manifest-path src-tauri/Cargo.toml <test_name>` — run a single backend Rust test
- `cargo test --manifest-path src-tauri/crates/otp/Cargo.toml` — run OTP crate tests
- `cargo test --manifest-path src-tauri/crates/otp/Cargo.toml <test_name>` — run a single OTP crate test

Example single-test command:
- `cargo test --manifest-path src-tauri/Cargo.toml normalizes_trailing_slashes_without_breaking_roots`

### Docs site
- `pnpm --dir docs-site start` — build and serve the docs site locally for all locales (`/` and `/en/`)
- `pnpm --dir docs-site start:zh` — run the zh-CN docs dev server with hot reload
- `pnpm --dir docs-site start:en` — run the English docs dev server with hot reload
- `pnpm --dir docs-site start:ko` — run the Korean docs dev server with hot reload
- `pnpm --dir docs-site build` — build the docs site

## Big-picture architecture

- This is a Tauri 2 desktop app: React/TypeScript frontend in `src/`, Rust backend in `src-tauri/src/`, and IPC between them via Tauri commands/events.
- The frontend should call Rust through the typed wrapper in `src/lib/invoke.ts`, not raw scattered `invoke()` calls where a shared wrapper already exists.
- Tauri commands are registered centrally in `src-tauri/src/lib.rs` and grouped by concern under `src-tauri/src/cmd/` (`session`, `sftp`, `connection`, `credential`, `settings`, `watcher`, `translate`, `stats`, `tunnel`, `proxy`, `otp`, `importer`, plus `app`, `backup`, `clipboard`, `cloud_sync`, `log`, and `ai`).

### Window model
- `src/main.tsx` decides between two boot paths:
  - main window: `AppProvider` + `App.tsx`
  - child windows: `ChildAppProvider` + `ChildWindowRouter`
- Child windows are opened from `src/lib/windowManager.ts` using `?window=` query params. Current child-window flows include settings, new-session, quick-command, and per-file auto-upload dialogs.
- Modal child-window focus/enable state is managed in `windowManager.ts` plus `src/ChildWindowRouter.tsx`; changes here affect cross-window UX.

### Frontend state model
- `src/context/AppContext.tsx` is the main state container. It owns:
  - tab/session workspace state
  - persisted UI/app settings
  - saved connections/groups loading and refresh
  - startup session restoration from persisted `ui.open_tabs`
- `src/context/ChildAppProvider.tsx` is a lightweight provider for child windows. It only loads/saves settings and emits cross-window events; it does not manage the full tab/session workspace.
- `src/context/TransferContext.tsx` separately tracks file transfer progress from backend `transfer-event` notifications.

### Workspace / terminal model
- The terminal workspace has two distinct layers that are easy to confuse:
  - `src/lib/workspaceTabs.ts` manages the persistent logical tab model. Each tab owns a recursive pane tree (`leaf` session panes and `split` panes). This is what gets serialized into `ui.open_tabs` for startup restore.
  - `src/lib/tabWindows.ts` manages the runtime terminal window layout: which tabs live in each split window leaf, active tab per leaf, and window split ratios for the multi-tab/multi-split UI.
- `src/App.tsx` is the shell that composes activity bars, left/right panels, the terminal workspace, quick command / serial send bottom panels, OTP dialogs, transfer UI, recording, and lock screen.
- `src/components/terminal/XTerminal.tsx` is the xterm.js integration point. It wires Fit/Search/WebLinks addons, shell integration, command suggestions, reconnect hooks, and listens to per-session backend events.

### Backend runtime model
- `src-tauri/src/lib.rs` constructs and stores the shared backend managers in Tauri state:
  - `SessionManager`
  - `TunnelManager`
  - `RecordingManager`
  - `PendingAuthManager`
  - `HostKeyVerifyManager`
  - `QuickCommandsStore`
  - `CloudSyncManager`
  - `AgentApprovalManager` (gates AI agent command execution)
- Tauri commands are registered centrally in `src-tauri/src/lib.rs`; newer backend capability areas now include app, backup, clipboard, cloud sync, logging, and AI in addition to sessions/SFTP/settings/importers.
- `src-tauri/src/core/session.rs` contains `SessionManager`, which is the central registry for active sessions, command routing, command history, fuzzy history search, and session lifecycle events.
- Session implementations live under `src-tauri/src/core/`:
  - `ssh/` for SSH transport, auth, OSC/CWD tracking, tunnels, and SFTP
  - `pty.rs` for local terminal sessions
  - `telnet.rs` for Telnet sessions
  - `serial.rs` for serial sessions
  - `recording.rs` for terminal recording
  - `watcher.rs` for file-watch driven flows
  - `importer.rs` for Xshell / MobaXterm / WindTerm import
  - `cloud_sync.rs` for sync/backup runtime and conflict events
  - `portable_snapshot.rs` for defining what sync/backup payloads include
  - `ai/` for provider calls, streaming responses, structured command cards, agent execution/approval, prompt redaction, and audit/history storage
- Backend session I/O is event-driven. The Rust side emits session-specific and app-wide events such as `terminal-output-{id}`, `cwd-changed-{id}`, `session-closed-{id}`, `sessions-changed`, `connections-changed`, `command-history-changed`, `transfer-event`, `otp-request`, `cloud-sync-status-changed`, `cloud-sync-history-changed`, and `cloud-sync-conflict`.

### SSH / auth / transfer details
- SSH logic is split across `src-tauri/src/core/ssh/`:
  - `client.rs` handles russh client setup, keepalive config, proxy-aware connection setup, and TOFU-style `known_hosts` verification (host-key prompts are coordinated through `HostKeyVerifyManager`)
  - `auth.rs` handles saved auth loading plus keyboard-interactive / OTP flows through `PendingAuthManager` and the `otp-request` event
  - `io.rs` streams terminal output and emits CWD updates
  - `sftp.rs` implements remote file operations and emits transfer progress events consumed by `TransferContext`
  - `tunnel.rs` manages local / remote / dynamic SSH tunnel behavior
- SFTP commands exposed to the frontend are in `src-tauri/src/cmd/sftp.rs`; the file explorer and transfer UI sit on top of these commands and events.
- File watcher / auto-upload flows bridge backend and child windows: remote files are downloaded locally, watched by `src-tauri/src/core/watcher.rs`, then uploaded back through the auto-upload UI flow.

### Persistence model
- App data lives under `~/.nyaterm/`, but the primary store is now `~/.nyaterm/nyaterm.redb` rather than a set of standalone JSON files.
- Important redb JSON documents include `settings`, `sessions`, `keys`, `passwords`, `otp`, `quick-command`, `tunnels`, `proxies`, `history`, `cloud-sync`, `cloud-sync-state`, `ai-history`, and `ai-audit`.
- Important text documents stored through the same layer include `known_hosts` and `master.key`.
- Sensitive values are encrypted before being written; cloud-sync credentials and other secret-bearing config need to stay within the existing crypto/storage helpers.
- When changing settings or workspace persistence, update both the frontend defaults (`AppContext` / `ChildAppProvider`) and the Rust persistence/migration code (`src-tauri/src/config/settings/mod.rs`, `src-tauri/src/config/ui.rs`, and related `src-tauri/src/storage.rs` / cloud-sync snapshot code when applicable).
- The app also contains legacy Dragonfly migration paths; if you change persistence formats, check that import/migration behavior still makes sense for `~/.dragonfly/` data.

## Project-specific guidance

- If a task touches UI, prefer shadcn/ui patterns and components. The repo has an explicit Cursor rule for this in `.cursor/rules/ui.mdc`.
- shadcn is configured in `components.json`, and shared UI components live in `src/components/ui/`.
- When changing user-facing UI text, update both locale files:
  - `src/i18n/locales/en.json`
  - `src/i18n/locales/zh-CN.json`
  - `src/i18n/locales/zh-TW.json`
  - `src/i18n/locales/ko.json`
- The root app currently has no dedicated frontend unit test runner configured in `package.json`; the automated tests in this repo are Rust tests under `src-tauri/` and `src-tauri/crates/otp/`.
- Frontend linting includes a no-`console` check before Biome (`pnpm lint` runs `scripts/check-no-console.mjs` and `biome check src/`).
- Vite uses the `@` alias for `src/`.
- Tauri dev/build behavior is configured in `src-tauri/tauri.conf.json`; the dev server runs on port `1420` with HMR on `1421`.
- There is a separate Docusaurus docs app in `docs-site/`. The most useful repo docs for implementation context are in `docs-site/docs/development/` (`architecture.md`, `frontend.md`, `backend.md`, `setup.md`, `contributing.md`).
- Repo docs and recent history use Conventional Commits (`feat:`, `fix:`, `perf:`, `chore:`).

---
> Source: [nyakang/nyaterm](https://github.com/nyakang/nyaterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
