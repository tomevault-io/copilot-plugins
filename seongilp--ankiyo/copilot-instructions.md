## ankiyo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

ankiyo is a Flutter spaced-repetition study app (Anki-style) targeting **web, Android, and iOS** from one codebase. It does two things Anki proper does not bundle together:

1. **Imports `.apkg` decks** — parses the embedded SQLite collection and renders Anki card templates to HTML.
2. **Generates decks from text via Claude** — extracts English vocabulary from a pasted passage (for Korean learners) into front/back cards.

Reviews are scheduled with a custom SM-2 hybrid scheduler.

## Commands

```bash
flutter pub get                       # install deps
flutter analyze                       # lint (flutter_lints, see analysis_options.yaml)
flutter test                          # run all tests
flutter test test/srs/scheduler_test.dart          # single test file
flutter test --name "graduates to review"          # single test by name

# Run. LLM features need either an API key or a proxy base URL passed via --dart-define.
flutter run -d chrome
flutter run -d <android-device-id>

# Builds
flutter build web
flutter build apk --release
```

### Runtime configuration (compile-time env, read in `lib/main.dart`)

Passed with `--dart-define`. Without them the app still runs; only LLM deck-generation is disabled.

- `ANTHROPIC_API_KEY` — direct key (optional when using the proxy)
- `ANTHROPIC_BASE_URL` — default `https://api.anthropic.com`; point at the Deno proxy to keep the key server-side
- `CF_AIG_TOKEN` — Cloudflare AI Gateway bearer token (only if routing through CF)

Example: `flutter run -d chrome --dart-define=ANTHROPIC_BASE_URL=https://<proxy>.deno.dev`

`.env.web` holds web run config locally (gitignored values).

## Architecture

Layered, with platform differences isolated behind conditional imports rather than runtime checks.

### Platform-conditional imports (the key pattern)

Two subsystems compile a different implementation for web vs native. The pattern is a stub file that does `export '..._native.dart' if (dart.library.io) ...` style conditional export. **When changing either, update all three files (interface + native + web) together.**

- **Database** — `lib/storage/db.dart` → `db_native.dart` (sembast_io, file under app documents dir) / `db_web.dart` (sembast_web, IndexedDB).
- **`.apkg` opening** — `lib/apkg/collection_opener.dart` → `collection_opener_native.dart` (writes bytes to a temp file, opens with `sqlite3`) / `collection_opener_web.dart` (loads `web/sqlite3.wasm` into an in-memory VFS).

### Layers

- **`lib/apkg/`** — `.apkg` (ZIP) → SQLite collection → study cards. `apkg_reader.dart` unzips and extracts `collection.anki21`/`.anki2` (the newer zstd `.anki21b` is rejected). `anki_db.dart` reads notes/cards/models/decks; `anki_model.dart`/`anki_note.dart` model the schema (fields split on `\x1f`). `card_builder.dart` renders Anki Mustache-ish templates (`{{field}}`, `{{#cond}}`, cloze `{{c1::…}}`) to front/back HTML.
- **`lib/srs/`** — `card_state.dart` (immutable `CardState`: status, due, intervalDays, easeFactor, reps, lapses, learning step) and `scheduler.dart` (SM-2 hybrid: learning steps → graduation → review with ease multiplier; lapses route through relearning). This is the core domain logic and is heavily unit-tested.
- **`lib/study/`** — `review_queue.dart` builds the ordered session queue (due learning/relearning first, then due reviews capped at `reviewLimit`, then new cards capped at `newLimit`).
- **`lib/storage/`** — sembast-backed persistence. `models.dart` (`Deck`, `StudyCard`), `deck_repository.dart` (CRUD), `review_log.dart` (per-rating history → stats + "weak card" detection), `settings_store.dart` (`SrsSettings`: new/review daily limits), `import_service.dart` (parsed deck → persisted deck with fresh UUIDs).
- **`lib/llm/`** — `anthropic_client.dart` (POSTs `/v1/messages` over `http`, configurable base URL, BYOK or proxy) and `word_extractor.dart` (passage → `ExtractedCard`s; defaults to a Haiku model).
- **`lib/ui/`** — Provider-based. Screens (`deck_list`, `review`, `import`, `create_deck`, `review_extracted`, `stats`, `settings`) backed by `ChangeNotifier` controllers in `lib/ui/controllers/`. `app.dart` wires the Provider tree and injects the Anthropic config from `main.dart`. `theme.dart` is Material 3 light/dark; `format_due.dart` renders human-readable due times.

### State management

`provider` + `ChangeNotifier`. Controllers hold session/screen state and call repositories; screens consume via `Consumer`/`context.watch`. Follow the immutable-update convention (`CardState` etc. return new copies — do not mutate in place).

## Backend proxies (`worker/`, `deno-proxy/`)

The Flutter app can talk to Anthropic directly, but to avoid shipping an API key in the client there is a thin forwarding proxy.

- **`deno-proxy/`** — active. `main.ts` is a Deno server exposing `/health` and `/v1/messages`, forwarding to Anthropic with the key from the `ANTHROPIC_API_KEY` env var (set in the Deno Deploy dashboard). Deploy via `deno-proxy/deno.json` tasks.
- **`worker/`** — a Cloudflare Worker variant, kept but deprecated (Anthropic blocks Cloudflare egress IPs). Prefer the Deno proxy.

## Tests

Mirror `lib/` under `test/` (`test/apkg`, `test/srs`, `test/storage`, `test/study`, `test/ui`). The scheduler, apkg parsing, and storage layers carry the bulk of coverage. Widget tests for `review_screen` use `runAsync` to avoid hangs from async timers.

## Android build

`android/` targets Java/Kotlin 17, `applicationId` `com.example.ankiyo`, SDK versions from the Flutter toolchain. No custom native code.

---
> Source: [seongilp/ankiyo](https://github.com/seongilp/ankiyo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
