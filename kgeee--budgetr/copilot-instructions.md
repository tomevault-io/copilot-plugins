## budgetr

> Two apps share this repo:

# budgetr — monorepo

Two apps share this repo:

- **`web/`** — the Next.js 16 + TypeScript app (SQLite via Drizzle, Plaid + Finnhub + Yahoo Finance). This is the original budgetr and the source of truth for business logic. It also serves as a thin backend proxy holding the Plaid/Finnhub secrets that can't ship in an app binary.
- **`apple/`** — the native SwiftUI app for macOS + iOS (Core Data + NSPersistentCloudKitContainer for private-database iCloud sync). Generated via XcodeGen from `apple/project.yml`. See `apple/README.md` and `SWIFT_MIGRATION_PLAN.md`.

## Working in `web/`

cd into `web/` first — that's where `package.json`, `node_modules`, and the env files live.

<!-- BEGIN:nextjs-agent-rules -->
### This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `web/node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

## Working in `apple/`

`brew install xcodegen` then `xcodegen generate` inside `apple/` to (re)build the Xcode project. The Core Data model is CloudKit-shaped — no unique constraints, all attributes optional/defaulted, relationships optional with inverses.

---
> Source: [kGeee/budgetr](https://github.com/kGeee/budgetr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
