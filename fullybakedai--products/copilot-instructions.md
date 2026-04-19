## products

> Product construction repo for FullyBakedAI. Shared across machines via git — used by both human sessions (MacBook) and Frank (Mac Studio).

# Products

Product construction repo for FullyBakedAI. Shared across machines via git — used by both human sessions (MacBook) and Frank (Mac Studio).

## Structure

Each product gets its own top-level folder (e.g. `Modulo/`). Inside each:
- `Kickoff/` — initial briefs and kickoff docs
- `Personas/` — user personas
- `Discovery/` — research and findings
- `ProductBrief/` — the product brief
- `Prototype/` — HTML prototypes
- Design system docs, context files, etc.

## Workflow

- Push when you're done working, pull when you start
- No auto-sync hooks — manual git only
- This repo is NOT the OpenClaw workspace — never symlink or merge them

## Strategic Context — READ BEFORE BUILDING

Strategic decisions, methodology, architecture patterns, and design system rules live in the **OpenClaw workspace** at `~/workspace/`. This is the single source of truth. Before starting any product work, read:

1. **`~/workspace/Platform/Docs/design-quality-brief.md`** — **READ FIRST.** Design principles, visual quality standards, token compliance rules, accessibility requirements. This is what "good" looks like.
2. **`~/workspace/CLAUDE.md`** — methodology rules, product names, architecture principles, build pipeline
3. **`~/workspace/Platform/BakeKit/Decisions/`** — all ADRs (component stack, design language, config-driven products, etc.)
4. **`~/workspace/Platform/Docs/building-on-bakekit.md`** — how to build products on BakeKit

These files are updated by the strategy/QA side of the operation. When they change, the rules change. Always read the current versions — don't cache assumptions from previous sessions.

**Do not duplicate strategic content into this repo.** If you need to reference a decision, point to the workspace file. If you think a decision is missing or wrong, flag it — don't create a local override.

## For Frank

Product files live at `~/products` on the Mac Studio. Reference them read-only — don't commit from agent sessions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FullyBakedAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
