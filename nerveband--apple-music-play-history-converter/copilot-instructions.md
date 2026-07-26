## apple-music-play-history-converter

> Apple Music Play History Converter is now a Tauri-based desktop app.

# Project Notes (Tauri-Only)

## Overview

Apple Music Play History Converter is now a Tauri-based desktop app.

- UI: React + TypeScript (`tauri-app/src`)
- Native host: Rust (`tauri-app/src-tauri`)
- Data/search/export runtime: Python sidecar (`tauri-app/python-sidecar/sidecar.py`)
- Shared backend modules: `src/apple_music_history_converter`

## Primary Commands

```bash
cd tauri-app && npm run tauri dev
cd tauri-app && npm run test
cd tauri-app/src-tauri && cargo check
cd tauri-app/python-sidecar && python3 test_sidecar.py
cd tauri-app/python-sidecar && python3 test_retry_rate_limited.py
```

## Python Dependencies

Use:

- `tauri-app/python-sidecar/requirements.txt`

## Important Paths

- `tauri-app/src/App.tsx`
- `tauri-app/src/hooks/useSearch.ts`
- `tauri-app/src/components/ResultsPanel.tsx`
- `tauri-app/src/components/ResultsTable.tsx`
- `tauri-app/src/components/sections/ServicesSection.tsx`
- `tauri-app/src/components/sections/DatabaseSection.tsx`
- `tauri-app/src-tauri/src/lib.rs`
- `tauri-app/src-tauri/src/sidecar.rs`
- `tauri-app/python-sidecar/sidecar.py`
- `src/apple_music_history_converter/music_search_service_v2.py`
- `src/apple_music_history_converter/musicbrainz_manager_v2_optimized.py`
- `src/apple_music_history_converter/apple_music_service.py`
- `src/apple_music_history_converter/export_formats.py`

## Current Architecture Rule

New development must target the Tauri app and its sidecar flow.

## Release Workflow

- Canonical maintainer release steps: `docs/RELEASING.md`
- Short operational checklist for agents: `AGENTS.md`
- Do not commit `release-artifacts/`
- Keep versions in sync across `tauri-app/package.json`, `tauri-app/src-tauri/Cargo.toml`, and `tauri-app/src-tauri/tauri.conf.json`

---
> Source: [nerveband/Apple-Music-Play-History-Converter](https://github.com/nerveband/Apple-Music-Play-History-Converter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
