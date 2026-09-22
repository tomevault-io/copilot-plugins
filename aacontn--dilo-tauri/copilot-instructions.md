## dilo-tauri

> This file provides guidance to AI coding assistants working with code in this repository.

# CLAUDE.md / AGENTS.md

This file provides guidance to AI coding assistants working with code in this repository.

> **`CLAUDE.md` and `AGENTS.md` are byte-identical copies, and `CLAUDE.md` is the
> source of truth.** Two files exist because different tools look for different
> names (Claude Code reads `CLAUDE.md`, Codex reads `AGENTS.md`); they are full
> copies rather than one pointing at the other because not every tool resolves a
> `Read @file` reference, and a tool that doesn't would end up with no
> instructions at all. **Edit `CLAUDE.md`, then copy it over `AGENTS.md`** —
> `tests/unit/agentInstructions.test.ts` fails if they drift apart.

> **Dilo** is a Spanish-first fork of [Handy](https://github.com/cjpais/Handy) (`upstream` remote). Product decisions live in `docs/superpowers/specs/`. Keep the Rust core close to upstream so `git merge upstream/main` stays cheap; brand/UI/default changes go in focused commits. All user-facing copy is Spanish-first (es locale is authored, not machine-translated — keep its voice: tuteo, direct, zero corporate filler).

## ⏸️ Estado: congelado en 0.3.2 (2026-09-20)

**Este repo no recibe más trabajo.** Dilo se reescribe como app nativa de Mac
en `/Volumes/SSD 1/Dilo/mac/` (Swift, fork de Talkify); la dirección está en
`docs/superpowers/specs/2026-09-20-dilo-mac-nativo-design.md`. Los specs de
producto de este repo siguen siendo la fuente de las decisiones (modos,
español, plataforma abierta, reuniones) y se portan leyéndolos, no leyendo
el Rust. Lo único que se toca aquí: el README de congelamiento, y specs o
bitácora mientras el repo nuevo no tenga los suyos. No proponer features,
arreglos ni merges con upstream.

## Dirección de producto — plataforma conversacional abierta

**Dilo es la pieza central open source**, no el cliente cautivo de un backend,
agente o producto particular. Su propósito es ser una interfaz conversacional
universal: recibe voz o texto, transcribe, conversa y presenta la respuesta por
voz, texto o ambos. Cada persona decide a qué modelo, asistente, automatización
o sistema conectarlo.

- Dilo sigue siendo plenamente útil sin conexiones externas: dictado,
  procesamiento local y voz local.
- Las capacidades externas entran por contratos genéricos y reemplazables:
  primero un destino asistente configurable; después conectores/adaptadores
  con permisos explícitos y, cuando corresponda, protocolos como MCP.
- No introducir en el núcleo nombres, reglas de negocio ni dependencias de un
  backend particular. Una instalación privada puede ser muy potente sin
  convertir esa implementación en la arquitectura del producto público.
- Dilo posee la experiencia conversacional —captura, STT, TTS, sesión,
  progreso y presentación—; el backend conectado interpreta, planifica y
  ejecuta. Dilo no se convierte en ERP, orquestador de agentes ni autoridad de
  negocio.
- La voz puede proponer una acción, pero nunca autentica ni autoriza una
  operación sensible. Los conectores declaran permisos y las credenciales no
  se guardan como texto plano en la configuración.
- El nombre/persona y el futuro wake word son configurables por cada usuario;
  ninguna personalidad concreta define el producto.

La dirección aprobada y sus límites están en
`docs/superpowers/specs/2026-07-22-dilo-plataforma-conversacional-abierta.md`.
Es una definición de producto, **no autorización para implementar conectores
sin su diseño técnico y plan correspondientes**.

La misma especificación incluye una nota del 2026-07-24 para explorar **Dilo
Online con Nova 2 Sonic**: wake word local y conversación cloud opcional con
Command Center y agentes. Sonic debe ser un proveedor reemplazable detrás de un
contrato conversacional propio; no reemplaza el dictado offline ni entra por el
conector de texto Bedrock Mantle. La nota no autoriza implementación.

## Development Commands

**Prerequisites:**

- [Rust](https://rustup.rs/) (latest stable)
- [Bun](https://bun.sh/) package manager

**Core Development:**

```bash
# Install dependencies
bun install

# Run in development mode
bun run tauri dev
# If cmake error on macOS:
CMAKE_POLICY_VERSION_MINIMUM=3.5 bun run tauri dev

# Build for production
bun run tauri build

# Frontend only development
bun run dev        # Start Vite dev server
bun run build      # Build frontend (TypeScript + Vite)
bun run preview    # Preview built frontend
```

**Linting and Formatting (run before committing):**

```bash
bun run lint              # ESLint for frontend
bun run lint:fix          # ESLint with auto-fix
bun run format            # Prettier + cargo fmt
bun run format:check      # Check formatting without changes
bun run format:frontend   # Prettier only
bun run format:backend    # cargo fmt only
```

**Model Setup (Required for Development):**

```bash
mkdir -p src-tauri/resources/models
curl -o src-tauri/resources/models/silero_vad_v4.onnx https://blob.handy.computer/silero_vad_v4.onnx
```

For detailed platform-specific build setup, see [BUILD.md](BUILD.md).

## Architecture Overview

Dilo is a cross-platform open conversational interface built with Tauri 2.x
(Rust backend + React/TypeScript frontend). Speech-to-text remains its base,
but the product direction also includes interchangeable assistant destinations
and spoken/text responses without coupling the core to any particular backend.

### Backend Structure (src-tauri/src/)

- `lib.rs` - Main entry point, Tauri setup, manager initialization
- `managers/` - Core business logic:
  - `audio.rs` - Audio recording and device management
  - `model.rs` - Model downloading and management
  - `transcription.rs` - Speech-to-text processing pipeline
  - `history.rs` - Transcription history storage
- `audio_toolkit/` - Low-level audio processing:
  - `audio/` - Device enumeration, recording, resampling
  - `vad/` - Voice Activity Detection (Silero VAD)
- `commands/` - Tauri command handlers for frontend communication
- `cli.rs` - CLI argument definitions (clap derive)
- `shortcut.rs` - Global keyboard shortcut handling
- `settings.rs` - Application settings management
- `overlay.rs` - Recording overlay window (platform-specific)
- `signal_handle.rs` - `send_transcription_input()` reusable function
- `utils.rs` - Platform detection helpers

### Frontend Structure (src/)

- `App.tsx` - Main component with onboarding flow
- `components/` - React UI components:
  - `settings/` - Settings UI
  - `model-selector/` - Model management interface
  - `onboarding/` - First-run experience
  - `overlay/` - Recording overlay UI
  - `update-checker/` - App update notifications
  - `shared/`, `ui/`, `icons/`, `footer/` - Shared components
- `hooks/useSettings.ts` - Settings state management hook
- `stores/settingsStore.ts` - Zustand store for settings
- `bindings.ts` - Auto-generated Tauri type bindings (via tauri-specta)
- `overlay/` - Recording overlay window entry point
- `lib/types.ts` - Shared TypeScript type definitions

### Key Architecture Patterns

**Manager Pattern:** Core functionality organized into managers (Audio, Model, Transcription) initialized at startup and managed via Tauri state.

**Command-Event Architecture:** Frontend → Backend via Tauri commands; Backend → Frontend via events.

**Pipeline Processing:** Audio → VAD → Whisper/Parakeet → Text output → Clipboard/Paste

**State Flow:** Zustand → Tauri Command → Rust State → Persistence (tauri-plugin-store)

### Technology Stack

**Core Libraries:**

- `transcribe-cpp` - Local Whisper-family inference (GGML/GGUF) with GPU acceleration
- `transcribe-rs` - ONNX speech recognition (Parakeet, Moonshine, SenseVoice, etc.)
- `cpal` - Cross-platform audio I/O
- `vad-rs` - Voice Activity Detection
- `rdev` - Global keyboard shortcuts
- `rubato` - Audio resampling
- `rodio` - Audio playback for feedback sounds

### Application Flow

1. **Initialization:** App starts minimized to tray, loads settings, initializes managers
2. **Model Setup:** First-run downloads preferred Whisper model (Small/Medium/Turbo/Large)
3. **Recording:** Global shortcut triggers audio recording with VAD filtering
4. **Processing:** Audio sent to Whisper model for transcription
5. **Output:** Text pasted to active application via system clipboard

### Settings System

Settings are stored using Tauri's store plugin with reactive updates:

- Keyboard shortcuts (configurable, supports push-to-talk)
- Audio devices (microphone/output selection)
- Model preferences (Small/Medium/Turbo/Large Whisper variants)
- Audio feedback and translation options

**One-time notices that must survive a restart (0.2.3):** the 0.2.3 migration
clears mode shortcuts this keyboard can't trigger, and it runs at startup, when
there is usually no window open — someone who dictates keeps Dilo in the tray.
The notice is therefore parked in the settings store under its own top-level key
(`pending_mode_shortcut_notice`, a sibling of `settings`, **not** a field of
`AppSettings` — that would change generated `src/bindings.ts`) and re-emitted as
the plain `mode-shortcuts-cleared` event from `utils::emit_window_shown` and
`commands::get_app_settings`. Rust never consumes it (it cannot know whether a
webview had already attached its listener); `App.tsx` shows it once per session
and `shortcut::change_mode_shortcut` deletes it once the person assigns a key.

### Single Instance Architecture

The app enforces single instance behavior — launching when already running brings the settings window to front rather than creating a new process. Remote control flags (`--toggle-transcription`, etc.) work by launching a second instance that sends args to the running instance via `tauri_plugin_single_instance`, then exits.

## Internationalization (i18n)

All user-facing strings must use i18next translations. ESLint enforces this (no hardcoded strings in JSX).

**Adding new text:**

1. Add key to `src/i18n/locales/en/translation.json`
2. Use in component: `const { t } = useTranslation(); t('key.path')`

**File structure:**

```
src/i18n/
├── index.ts           # i18n setup
├── languages.ts       # Language metadata
└── locales/
    ├── en/translation.json  # English (source)
    ├── de/, es/, fr/, ja/, ru/, zh/, ...
    └── ...
```

For translation contribution guidelines, see [CONTRIBUTING_TRANSLATIONS.md](CONTRIBUTING_TRANSLATIONS.md).

## Code Style

**Rust:**

- Run `cargo fmt` and `cargo clippy` before committing
- Handle errors explicitly (avoid unwrap in production)
- Use descriptive names, add doc comments for public APIs

**TypeScript/React:**

- Strict TypeScript, avoid `any` types
- Functional components with hooks
- Tailwind CSS for styling
- Path aliases: `@/` → `./src/`

## CLI Parameters

Dilo supports command-line parameters on all platforms for integration with scripts, window managers, and autostart configurations.

**Implementation:** `cli.rs` (definitions), `main.rs` (parsing), `lib.rs` (applying), `signal_handle.rs` (shared logic)

| Flag                             | Description                                                    |
| -------------------------------- | -------------------------------------------------------------- |
| `--toggle-transcription`         | Toggle recording on/off on a running instance                  |
| `--toggle-post-process[=<mode>]` | Toggle recording with a transformation mode on/off (see below) |
| `--cancel`                       | Cancel the current operation on a running instance             |
| `--start-hidden`                 | Launch without showing the main window (tray icon visible)     |
| `--no-tray`                      | Launch without system tray (closing window quits the app)      |
| `--debug`                        | Enable debug mode with verbose (Trace) logging                 |

**Key design decisions:**

- CLI flags are runtime-only overrides — they do NOT modify persisted settings
- Remote control flags work via `tauri_plugin_single_instance`: second instance sends args, then exits
- `send_transcription_input()` in `signal_handle.rs` is shared between signal handlers and CLI

**`--toggle-post-process` and `SIGUSR1` (0.2.3):** there is no "active mode"
anymore — a transformation mode comes from the shortcut that started the
dictation (`mode:<id>`). Both entry points now resolve a real mode through
`signal_handle::resolve_post_process_target()`: the optional value picks it by
id or by name (`--toggle-post-process=dilo-code`,
`--toggle-post-process="Correo"`), and without a value (always, for `SIGUSR1`,
which carries no arguments) Dilo uses the **first mode that has a key
assigned** — the same one that key would run. If nothing resolves, it logs and
records nothing instead of pasting raw dictation under a promise of
transforming it. The `transcribe_with_post_process` binding is no longer
triggered by anything.

## Debug Mode

Access debug features: `Cmd+Shift+D` (macOS) or `Ctrl+Shift+D` (Windows/Linux)

## Platform Notes

- **macOS**: Metal acceleration, accessibility permissions required for keyboard shortcuts
- **Windows**: Vulkan acceleration, code signing
- **Linux**: OpenBLAS + Vulkan, limited Wayland support, overlay uses GTK layer shell (disable with `DILO_NO_GTK_LAYER_SHELL=1`; the legacy `HANDY_NO_GTK_LAYER_SHELL` is still honored)

## Troubleshooting

See the [Troubleshooting](README.md#troubleshooting) section in README.md.

## GitHub workflow

This fork has no feature freeze: product direction is set by the maintainer (see the spec in `docs/superpowers/specs/`). Bug fixes that also apply upstream should ideally be contributed to [Handy](https://github.com/cjpais/Handy) following their contribution rules, then merged back here via `git fetch upstream && git merge upstream/main`.

- **Translations:** Follow [CONTRIBUTING_TRANSLATIONS.md](CONTRIBUTING_TRANSLATIONS.md). The `es` locale is hand-authored brand copy — do not machine-regenerate it.
- **Commits:** Use conventional commit prefixes (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`). Focus the message on _why_, not _what_.

---
> Source: [aacontn/dilo-tauri](https://github.com/aacontn/dilo-tauri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
