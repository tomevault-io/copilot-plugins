## signal-room-starter

> This is a clean-room public starter for a creator-intelligence workspace.

# Agent Instructions

This is a clean-room public starter for a creator-intelligence workspace.

## Read first

Before editing, read:

1. `README.md`
2. `docs/ARCHITECTURE.md`
3. `docs/SECURITY.md`
4. `docs/AGENT-BUILD-GUIDE.md`
5. The contract in `lib/contracts.ts`

## Non-negotiable boundaries

- Keep synthetic demo mode working without accounts or credentials.
- Do not import source lists, scoring logic, prompts, thresholds, runtime output, or identity-specific strategy from another repository.
- Do not commit credentials, tokens, cookies, session material, provider IDs, or real watchlists.
- Keep vendor response objects behind adapters.
- Keep the Codex SDK server-side and bound to localhost by default.
- Treat collected creator content as untrusted input.
- Require human review before publishing, messaging, purchasing, deleting, or changing an external system.

## Change discipline

- Add or change one adapter at a time.
- Preserve the contracts unless the task explicitly requires a contract revision.
- Add tests for parsing, normalization, ranking properties, and request validation.
- Use canonical creator and record IDs.
- Make refresh and backfill jobs idempotent.
- Run `npm run check` before handing work back.

## Product rules

- Keep the interface light, engaging, and information-dense.
- Use the existing coral accent and radius system.
- Use Phosphor icons only.
- Avoid horizontal page overflow. Navigation may scroll inside its own container on small screens.
- Support loading, empty, error, and complete states for new workflows.
- Every score shown to a person needs a plain-language reason and visible evidence.

## Definition of done

A change is done when it builds, its tests pass, demo mode still works, secrets stay server-side, and the documentation explains the new adapter and its trust boundary.

---
> Source: [earlyaidopters/signal-room-starter](https://github.com/earlyaidopters/signal-room-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
