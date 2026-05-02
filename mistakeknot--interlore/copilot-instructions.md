## interlore

> Philosophy observer plugin. Scans decision artifacts (brainstorms, PRDs, flux-drive outputs, plans) to detect recurring design patterns and philosophy drift. Proposes PHILOSOPHY.md updates with evidence links.

# AGENTS.md — interlore

Philosophy observer plugin. Scans decision artifacts (brainstorms, PRDs, flux-drive outputs, plans) to detect recurring design patterns and philosophy drift. Proposes PHILOSOPHY.md updates with evidence links.

**Plugin Type:** Claude Code skill plugin
**Plugin Namespace:** `interlore`
**Current Version:** 0.1.0

## Canonical References
1. [`MISSION.md`](../../MISSION.md) — project mission.
2. [`PHILOSOPHY.md`](../../PHILOSOPHY.md) — design bets and principles (what interlore observes and proposes updates to).

## Architecture

interlore has three layers:
1. **Artifact discovery** — glob patterns from interpath source catalog
2. **Pattern extraction** — content-based tradeoff detection + Alignment/Conflict enrichment
3. **Proposal management** — structured YAML staging with accept/reject/defer lifecycle

State: `.interlore/proposals.yaml` at project root.

---
> Source: [mistakeknot/interlore](https://github.com/mistakeknot/interlore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
