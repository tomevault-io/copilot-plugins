## pagenest

> This file applies only to the PageNest repository and overrides unrelated rules

# PageNest Engineering Guide

This file applies only to the PageNest repository and overrides unrelated rules
from parent workspaces.

## Scope

- Keep the browser extension, local Windows service, and Obsidian viewer as
  three small, coordinated components.
- Preserve the local-first, single-file `.pagenest` workflow and legacy
  `.hermes` compatibility.
- Do not add a frontend framework, database, task queue, telemetry, or cloud
  service for a local fix.

## Code boundaries

- Generic browser behavior belongs in `extension/core/`; site behavior belongs
  in `extension/adapters/` behind the shared adapter contract.
- All ordinary service downloads go through
  `local-server/collector/network.py` and its SSRF/size checks.
- Keep sanitizing, rendering, media handling, storage, security, and optional
  organization separate.
- Store-extension pairing must stay disabled unless exact Chromium extension
  IDs are configured through `PAGENEST_EXTENSION_IDS`.

## Verification

Before delivery, run:

```powershell
local-server\.venv\Scripts\python -m pip check
local-server\.venv\Scripts\python scripts\validate_repository.py
local-server\.venv\Scripts\python -m ruff check local-server/collector local-server/vision_probe.py scripts
local-server\.venv\Scripts\python -m coverage run -m pytest -q
local-server\.venv\Scripts\python -m coverage report
```

Run `node --check` for every JavaScript file and every `tests/test_*.js` file.
Interaction changes also require a real browser or Obsidian check. Packaging
changes require the release and installer smoke tests.

## Release safety

- Never package or commit `.env`, tokens, vault data, logs, profiles, databases,
  collected pages, screenshots outside `store/assets/`, or media caches.
- Build store archives only through `scripts/package_store.py` and release
  archives only through `scripts/package_release.py`.
- Do not create a remote, push, publish a store item, or publish a Release
  without explicit user approval.
- Keep commits focused and independently testable. Do not reset, clean, stash,
  or overwrite user work.

---
> Source: [WYuHuai/PageNest](https://github.com/WYuHuai/PageNest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
