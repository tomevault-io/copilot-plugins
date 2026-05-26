## hicalendar

> - `HiCalendar/`: SwiftUI app source (views, managers, models, assets).

# Repository Guidelines
#用中文
#每次调整后，自动编译一次看看是否有问题，模拟器是iphone16
## Project Structure & Modules
- `HiCalendar/`: SwiftUI app source (views, managers, models, assets).
- `HiCalendar.xcodeproj/`: Xcode project configuration.
- `HiCalendarTests/` and `HiCalendarUITests/`: XCTest unit/UI tests.
- `Docs/` and `*.md`: setup and product docs (e.g., PRD.md, PUSH_SETUP_GUIDE.md).
- `supabase/` and `*.sql`: database schema, cron, and setup scripts.
- Scripts: Python utilities for APNS/Supabase (`*.py`, `manual_push.sh`).
- Secrets: APNS keys (`*.p8`, `apns_private_key.pem`) are local-use only—do not add new secrets.

## Build, Test, and Development
- Open in Xcode: `open HiCalendar.xcodeproj` (run app on a simulator/device).
- Build (CLI): `xcodebuild -project HiCalendar.xcodeproj -scheme HiCalendar -destination 'platform=iOS Simulator,name=iPhone 15' build`.
- Test (CLI): `xcodebuild -project HiCalendar.xcodeproj -scheme HiCalendar -destination 'platform=iOS Simulator,name=iPhone 15' test`.
- Python scripts: `python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt` then `python simple_push_test.py` or `./manual_push.sh`.

## Coding Style & Naming
- Language: Swift (SwiftUI).
- Indentation: 4 spaces; keep lines reasonably short (~120 cols).
- Naming: PascalCase for types (`EventStorageManager`), camelCase for vars/functions, `View`/`Manager` suffixes match current files.
- Structure: one primary type per file; group related helpers near usage.
- Formatting: use Xcode’s “Editor > Structure > Re-Indent” before committing.

## Testing Guidelines
- Framework: XCTest in `HiCalendarTests/` and `HiCalendarUITests/`.
- Names: test files `TypeNameTests.swift`; methods start with `test...`.
- Scope: add/adjust tests with each feature or bugfix; prefer deterministic tests (stub network where possible).
- Run: via Product > Test in Xcode or the `xcodebuild ... test` command above.

## Commit & Pull Requests
- Commits: imperative, concise subject (e.g., `feat: add event caching`). Include rationale in body if non-trivial.
- PRs: clear description, linked issues, screenshots/GIFs for UI changes, test plan steps, and updated docs (PRD/Docs) when applicable.
- CI/Checks: ensure local build and tests pass; avoid committing secrets or transient files.

## Security & Configuration
- Follow APPLE_AUTH_SETUP.md, PUSH_SETUP_GUIDE.md, and setup_apns_env.md for auth/push.
- Store credentials in keychain/env, not in VCS. For Supabase changes, apply SQL from `supabase/` carefully and document migrations.

---
> Source: [JerryChan0623/HiCalendar](https://github.com/JerryChan0623/HiCalendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-26 -->
