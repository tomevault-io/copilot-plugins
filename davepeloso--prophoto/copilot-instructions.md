## prophoto

> - If a requested change would cross package boundaries, do not implement a shortcut. Ask which package should own it


# ProPhoto Core Rules

- If a requested change would cross package boundaries, do not implement a shortcut. Ask which package should own it

## Architecture Ownership

- prophoto-assets = canonical media truth (Asset Spine)
- prophoto-booking = sessions / operational calendar truth
- prophoto-ingest = matching + assignment decisions
- prophoto-intelligence = derived intelligence only
- prophoto-contracts = shared DTOs, enums, events

## Non-Negotiables

- Intelligence MUST NOT mutate asset truth
- Booking MUST NOT own asset data
- Ingest owns decisions, not booking or assets
- Use events/contracts for cross-package communication
- Do not bypass manual locks with automated logic
- Prefer append-only history where defined

## Code Behavior

- Keep listeners thin
- Keep planners pure (no DB access)
- Use enums, not loose strings
- Do not introduce cross-package shortcuts

## Task Behavior

When making changes:

1. Identify owning package first
2. Do not broaden scope
3. Add tests for behavior changes
4. Preserve package boundaries

## System Loop (Current State)

Ingest -> SessionAssociationResolved  
-> Assets persist projection  
-> AssetSessionContextAttached  
-> Intelligence orchestrator

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davepeloso) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
