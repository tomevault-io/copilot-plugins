## even-toolkit

> You are working on the shared toolkit for Even Realities G2 apps.

# AGENTS.md - even-toolkit

You are working on the shared toolkit for Even Realities G2 apps.

## Product Role

`even-toolkit` is the source of truth for glasses display primitives, bridge helpers, page layout, voice routing, device info, and location helpers used by the Even apps.

Primary dependents right now:

- `even-browser`
- `even-ghub`
- `open-vide-g2`
- generated apps from `create-even-app`

## Current Closure Goal

Make the toolkit stable enough that downstream apps can depend on it without local hacks.

Focus areas:

- Keep `main` page mode as the current API; do not reintroduce legacy `home` naming.
- Keep display builder APIs deterministic and easy to test.
- Preserve G2 display constraints and small-screen readability.
- Keep voice/device/location helpers optional and browser-safe.
- Add tests/gates before broad refactors.

## Commands

Use the local package manager already present in the repo.

```bash
npm run build
```

If you add test infrastructure, wire it through `npm test` and keep `npm run build` green.

## Rules

- Do not commit generated `dist/`, logs, credentials, or local device captures.
- Keep changes additive unless a ticket explicitly asks for a migration.
- If a change affects downstream API, update README/migration notes in the same ticket.

---
> Source: [fabioglimb/even-toolkit](https://github.com/fabioglimb/even-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
