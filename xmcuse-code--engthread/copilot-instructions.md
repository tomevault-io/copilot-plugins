## engthread

> EngThread is a local-first engineering event timeline and work tracker. It is not a controlled production, quality-disposition, safety, or regulatory system.

# EngThread repository instructions

## Product boundary

EngThread is a local-first engineering event timeline and work tracker. It is not a controlled production, quality-disposition, safety, or regulatory system.

- Keep runtime workspace data in the browser. Do not add telemetry, analytics, remote uploads, authentication, or cloud storage without an approved architecture change.
- Use synthetic examples only. Never add employer-owned processes, company identifiers, customer information, production data, real backups, Excel exports, credentials, or personal names.
- Treat `app/lib/storage.ts`, `app/lib/types.ts`, and `app/lib/records.ts` as persistence-critical code.
- Do not rename the IndexedDB database, backup format, or legacy compatibility keys without a tested forward migration.
- Persisted business-field changes must retain revision history and a user-supplied reason. Migrations must preserve a pre-migration snapshot.
- Roadmap cards must continue to edit requirement records rather than duplicate source data.

## Development workflow

- Use Node.js 22.23.2 and `npm ci`.
- Run `npm run check` before submitting a pull request.
- Add visible static text to all three dictionaries: `en`, `zh-CN`, and `zh-TW`.
- Test visible changes at a desktop viewport and near 911×512 CSS pixels.
- Storage changes require an old-database or old-backup migration test.
- Packaging changes require archive-content and checksum verification; do not commit `release/`, `dist/`, bundled runtimes, or test artifacts.

## Scope discipline

Prefer the smallest general engineering workflow that solves a recurring problem. Open an issue before adding a new module, changing the data model, or introducing a network dependency.

---
> Source: [xmcuse-code/engthread](https://github.com/xmcuse-code/engthread) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
