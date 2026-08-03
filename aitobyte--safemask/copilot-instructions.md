## safemask

> Guidance for Claude Code (claude.ai/code) when working in this repository.

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

> **Authority note.** `AGENTS.md` at the repo root remains the day-to-day operator handbook and takes precedence for command flags, session notes, and short-lived context. This file focuses on stable architectural facts and conventions.

---

## Project Overview

**SafeMask** is a Tauri v2 desktop application for industrial-grade local privacy masking, version **2.0.0**. All processing — rules, regex, AI NER, records — runs **100% offline**. There are no telemetry or remote reporting paths in the codebase.

- **Backend**: Rust 2024 edition, Tauri v2, ONNX Runtime (`ort` 2.0.0-rc.12), Rayon, mmap
- **Frontend**: React 19 + TypeScript + Zustand + Tailwind v3 + Vite 6 (no routing library — tab state in Zustand)
- **AI model**: `openai/privacy-filter` q4-quantized ONNX, auto-discovered under `src-tauri/models/privacy-filter/` (multiple search paths, see below)
- **Distribution**: Windows (MSI + portable ZIP), macOS (DMG, arm64), Linux (AppImage + deb) via `tauri-action` on `v*` tag push

The frontend stack is **not** Vue 3 / Pinia. Any older document that says so is out of date.

---

## Development Commands

```bash
# Install dependencies
npm install

# Full-stack dev (Vite + Tauri window)
npm run tauri dev

# Frontend-only dev server (127.0.0.1:18924, strictPort)
npm run dev

# Frontend typecheck + build (tsc, NOT vue-tsc)
npm run build

# Production bundle
npm run tauri build
```

**Rust commands — run from the repo root, not `src-tauri/`.** The repo is a Cargo workspace (resolver 2), and `src-tauri` is a member named `SafeMask`.

```bash
cargo check   -p SafeMask
cargo fmt     -p SafeMask
cargo clippy  -p SafeMask -- -D warnings
cargo test    -p SafeMask
cargo test    -p SafeMask test_name -- --nocapture
```

- Rust tests live **inline** in `#[cfg(test)]` modules — there is no `tests/` directory.
- `.cargo/config.toml` currently pins an HTTP proxy at `127.0.0.1:7890`. CI strips it (`rm -rf .cargo/config*`); if you don't run that proxy locally, either remove/comment the file or override with your own.

---

## Repository Layout

```
SafeMask/                        # Cargo workspace root
├── src/                         # React 19 frontend
│   ├── App.tsx                  # Root component (tab switch, event subscriptions, lazy pages)
│   ├── main.tsx                 # ReactDOM.createRoot entry, imports style.css
│   ├── components/
│   │   ├── dashboard/           # FileProcessor, StatCard
│   │   ├── feedback/            # MagicFeedback
│   │   ├── history/             # HistoryList, DocumentPreview (UTF-8 byte→char highlight)
│   │   ├── layout/              # Sidebar, Header
│   │   ├── overlay/             # ExitConfirm
│   │   ├── rules/               # RuleManager (regex sandbox, debounced)
│   │   ├── settings/            # SettingsPage, ModelDownloadCard
│   │   └── ui/                  # Atomic primitives (mostly under-used, see §Notes)
│   ├── hooks/
│   │   ├── useAppStore.ts       # Zustand store (bootstrap 2-phase load)
│   │   ├── useTauriEvents.ts    # Generic listen() wrapper with StrictMode-safe cleanup
│   │   ├── useAudioFeedback.ts  # Web Audio oscillator SFX (single AudioContext)
│   │   └── useModelDownloader.ts# Standalone Zustand store for AI model download
│   ├── services/api.ts          # Typed IPC wrappers (`MaskAPI`)
│   ├── lib/                     # utils, maskColors
│   ├── style.css                # Tailwind + custom keyframes (no framer-motion at runtime)
│   └── vite-env.d.ts            # __APP_VERSION__ global
├── src-tauri/                   # Rust backend (workspace member "SafeMask")
│   └── src/
│       ├── main.rs              # Binary entry — Tauri setup, plugins, invoke_handler
│       ├── lib.rs               # Library entry (safemask_lib) — staticlib/cdylib/rlib
│       ├── ai_downloader.rs     # Model download pipeline (zip fetch + verify + extract)
│       ├── api/                 # #[tauri::command] IPC handlers
│       │   ├── files.rs         # process_file_gui
│       │   ├── system.rs        # rules CRUD, history, settings, AI toggle, engine info
│       │   └── text.rs          # mask_text
│       ├── common/              # AppState, AppError (thiserror), event constants, EntitySpanBrief
│       ├── core/                # Pure business logic — zero Tauri imports, independently testable
│       │   ├── hybrid_engine.rs # Main engine (Registry + Resolver + MaskingEngine)
│       │   ├── engine.rs        # Legacy MaskEngine (superseded — candidate for removal)
│       │   ├── recognizer/      # aho_corasick / regex / ner / checksum / context_enhancer + registry
│       │   ├── resolver/        # Sub-span carving conflict resolution
│       │   ├── masking/         # 6 strategies: Replace/PartialMask/Hash/Redact/Token/Template
│       │   ├── orchestrator/    # SceneMode (Shadow/Sentry) business layer
│       │   ├── rules.rs         # Rule + RuleGroup types
│       │   ├── config.rs        # AppSettings
│       │   └── download_auth.rs # HMAC-SHA256 device-fingerprint download tokens
│       ├── infra/               # OS interactions
│       │   ├── ai/              # ModelManager (state), NerEngine (ort inference)
│       │   ├── clipboard/       # monitor (600ms poll), handler, magic_paste (Alt+V)
│       │   ├── config/          # loader (YAML rules + settings), shortcut_manager
│       │   ├── fs/              # processor (mmap + rayon + 8MB chunk + 32-concurrent backpressure)
│       │   └── record_writer/   # Async Markdown audit-record writer (150 records/file, 5s/10-item flush)
│       ├── rules/               # Built-in rule YAML (~21 rules across auth/network/personal/code)
│       ├── custom/              # User-added rules
│       ├── capabilities/        # Tauri capability declarations
│       └── models/privacy-filter/  # ONNX model directory (not checked in)
├── AGENTS.md                    # Operator handbook (authoritative for session/tooling notes)
├── CLAUDE.md                    # This file
├── tsconfig.json                # strict:true, "@/*" → "./src/*"
├── vite.config.ts               # Dev server 127.0.0.1:18924, manualChunks, __APP_VERSION__ define
├── tailwind.config.js
└── Cargo.toml                   # Workspace root
```

---

## Architecture — the six-layer pipeline

```
Frontend (React 19)
   │  invoke() over Tauri IPC
   ▼
api/*.rs           (#[tauri::command] surface)
   │
   ▼
core/orchestrator  (SceneMode: Shadow vs Sentry, ClipboardProcessResult)
   │
   ▼
core/hybrid_engine (glue between recognizer registry, resolver, masking)
   │
   ├──▶ core/recognizer/*        (AC dictionary, byte regex, NER, checksum, context enhancer)
   │       ▲
   │       └── registry.rs runs non-context recognizers first, then context ones with `previous_spans`
   │
   ├──▶ core/resolver            (sub-span carving + container swallow + fragment prune)
   │
   └──▶ core/masking             (strategy dispatch by EntityType)
   │
   ▼
infra/*            (clipboard monitor, mmap file processor, record writer, ONNX loader, HMAC downloader)
```

Key invariants:

- `core/` has **zero** Tauri imports. It can be exercised entirely from `cargo test -p SafeMask`.
- Offsets throughout the recognizer and resolver layers are **byte offsets** into `&[u8]`, not char indices. Regex runs on `regex::bytes::Regex` with `unicode(false)` by default, and `\b` behaviour is intentional (see the many UTF-8-boundary tests in `core/engine.rs`).
- The resolver uses **sub-span carving**: when a high-priority narrow span overlaps a low-priority wide span, the wide one is sliced around the narrow one instead of being dropped. `test_carving_three_way` demonstrates chained slicing.
- Same-type AI spans that overlap a rule-source span of the same `EntityType` are **fully suppressed** (see `resolver::mod::rs` step 2.5) to prevent `[URL]<URL>[URL]` fragmentation.
- `HybridEngine::mask_line` returns `Cow::Borrowed` on the no-match fast path — do not break this zero-copy contract.

---

## Frontend architecture

- **State**: two independent Zustand stores. `useAppStore` holds settings, rules, history, engine info, active tab. `useModelDownloader` owns AI-model download state machine and its own event unsubscribers.
- **Bootstrap**: two-phase load in `useAppStore.bootstrap()`.
  - Phase 1 (`Promise.all` awaited): `getSettings`, `getStats` — dashboard first paint.
  - Phase 2 (`setTimeout(..., 100)`, not awaited): `getHistory`, `getAppInfo`, `getAiEngineStatus`, `getEngineInfo`.
- **Lazy pages**: `HistoryList`, `RuleManager`, `SettingsPage`, `ExitConfirm` are `React.lazy` + `Suspense`, with 500ms post-mount prefetch to warm the chunks.
- **Animation**: `framer-motion` is intentionally **not** used at runtime. All motion is CSS keyframes / Tailwind transitions. (`framer-motion` still appears in `package.json` — it is unused and can eventually be dropped.)
- **Fonts**: system font stack via `system-ui`. No self-hosted or Google-loaded webfonts.
- **Event subscription**: use `useTauriEvent<T>(name, cb)` from `hooks/useTauriEvents.ts`. It handles StrictMode double-mount and late-arriving unlisten safely.
- **IPC contract**: prefer adding methods to `services/api.ts::MaskAPI`. Two current downloader commands (`check_model_file`, `start_model_download`, `cancel_model_download`) bypass this layer and are invoked directly from `useModelDownloader`.
- Tab routing is a Zustand string — there is no `react-router` or equivalent.

---

## IPC flow

```
Frontend  →  invoke("cmd_name", { paramInCamelCase })
          →  #[tauri::command] async fn cmd_name(...) in api/
          →  (optionally) core/ or infra/ call
          →  Result<T, AppError> serialised to the frontend
```

Tauri auto-maps JS camelCase parameter names to Rust snake_case, so a JS `{ newSettings }` binds to `new_settings: AppSettings`. Keep this in mind when adding commands.

**Command registration is done twice**: declare the function under `api/*.rs`, then list it in `main.rs::main()` inside `tauri::generate_handler![...]`. Forgetting the second step compiles but the frontend gets `command not allowed`.

---

## AI engine

- Discovery order for the model directory (see `main.rs::find_models_dir`):
  1. `<exe_dir>/models/…` (portable / installed)
  2. `<cwd>/models/…` (dev)
  3. `<app_local_data_dir>/models/…` (legacy install)
  4. `<resource_dir>/models/…`
  5. Fallback: `<exe_dir>/models/privacy-filter/` (used as the download target)
- A directory is considered valid via `validate_model_dir` — it looks for either `model.onnx` or `model_q4.onnx`, plus `tokenizer.json`.
- The model is **not** required for the app to run. If it is missing, `HybridEngine::enable_ai_engine` logs a skip and the rule-based recognizers continue to work.
- `NerEngine::load` is called on a background thread by `NerRecognizer::ensure_loaded` — the first `analyze()` returns empty while loading proceeds; state transitions live on `ModelManager`.
- Thread limits: Rayon defaults to **2 threads** (`SAFEMASK_THREADS` env var overrides). ONNX Runtime intra/inter threads are set directly in `NerEngine::load` (not by `ORT_NUM_THREADS`).
- Global allocator is `mimalloc` (`#[global_allocator]` in `main.rs`).

---

## Rules and configuration

- Built-in YAML rules live in `src-tauri/rules/` under domain folders (`auth/ai`, `auth/database`, `network`, `personal`, `code`). They are compiled into the binary at package time via Tauri `resources`.
- User rules live in `src-tauri/custom/user_rules.yaml`, editable at runtime through `RuleManager`.
- Two YAML shapes are accepted by `ConfigLoader::parse_file`: a `RuleGroup { group, rules: [...] }` wrapper, or a bare `Vec<Rule>`.
- `AppSettings` (in `core/config.rs`) fields you'll actually touch:
  - `magic_paste_shortcut` (default `"Alt+V"`)
  - `shadow_mode_enabled` (default true)
  - `paste_delay_ms` (default 150)
  - `enable_visual_feedback`, `enable_audio_feedback`
  - `model_download_urls` (not serialised — deserialisation defaults from `default_model_urls()`)
  - `record_writer_enabled` (default false — audit records write plaintext PII to disk, opt-in only)
- Global shortcuts are (re)registered by `ShortcutManager::reload_magic_shortcut`. `Alt+M` is hard-registered on setup as the Shadow/Sentry mode toggle.

---

## Conventions

- **Rust edition 2024** is unusual — expect `let _ = expr;` for intentional discards, `parking_lot::{Mutex, RwLock}` over `std::sync`, `unsafe extern "system" fn` blocks for direct DWM FFI on Windows.
- Never hold a `parking_lot` guard across `.await`. Clone `Arc`s out of the guard first (existing code has explicit comments where this matters, e.g. `system.rs`).
- Prefer `regex::bytes` and byte-offset arithmetic over `char` iteration inside the recognizer/resolver layers.
- Tauri capabilities are declared in `src-tauri/capabilities/default.json`. If a new command needs a plugin, add both a `plugin(...)` line in `main.rs` and the capability entry.
- The Cargo proxy in `.cargo/config.toml` is treated as build-time local infrastructure. CI removes it; contributors without a local proxy should do the same.
- CI (`.github/workflows/release.yml`) triggers **only on tag push `v*`** and produces a draft release across macOS/Linux/Windows. There is no PR-time CI at present.

---

## Notes for iteration work

A few things worth knowing before you dive in — none of these are blocking, but they change how you'd approach related edits:

- `core/engine.rs::MaskEngine` is the pre-`HybridEngine` implementation. `HybridEngine` fully supersedes it; keep new work on `HybridEngine`.
- Under `src/components/ui/`, only `Toggle` and `SettingToggle` are actually consumed. `Badge`, `Button`, `Card`, `GlassPanel`, `EmptyState`, `Input` exist but are unused — many business components hand-roll the same styles. If you touch a page, prefer promoting those atoms rather than adding one more inline copy.
- File processing (`api/files.rs::process_file_gui`) trusts the incoming `input_path` string — validate before writing derived output files if you extend this path.
- `HashStrategy` currently returns the same `DefaultHasher`-based digest regardless of the `use_sha256` config flag. The switch exists but the branch is empty.
- If you add a new event name, put the constant in `common/events.rs` (`AppEvents::…`) rather than string-literal-ing it in both Rust and TS.

---

## Reference commands

| Task | Command |
|------|---------|
| Full dev environment | `npm run tauri dev` |
| Frontend only | `npm run dev` |
| Type-check + build frontend | `npm run build` |
| Rust check | `cargo check -p SafeMask` |
| Rust lint | `cargo clippy -p SafeMask -- -D warnings` |
| Rust tests | `cargo test -p SafeMask` |
| Rust format | `cargo fmt -p SafeMask` |
| Release bundle | `npm run tauri build` |

Today's active branch is `main`, main-branch releases are tag-driven (`git tag v2.x.y && git push --tags`).

---
> Source: [AiToByte/SafeMask](https://github.com/AiToByte/SafeMask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
