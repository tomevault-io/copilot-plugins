## gnosis

> Cursor operates as a constrained executor inside Gnosis.

# Gnosis Cursor Rules — Authoritative (Post–Milestone 2)

Cursor operates as a constrained executor inside Gnosis.
Governance, evidence, and artifacts are authoritative.

================================================================
PRE-CODE WORKFLOW (MANDATORY — NO EXCEPTIONS)
================================================================

Before ANY code changes:

1. Ensure GNOSIS_PROJECT_ID is set (required locally).
2. Run:
   npm run gnosis:preflight
3. Follow:
   docs/SOP_CURSOR_PLAYBOOK.md
   for artifact interpretation and required decision steps.
4. NO PUSH unless:
   CI=true npm run ci:fast:local
   passes cleanly.

Local enforcement is stricter than CI:
- If graph inspection is required and UNAVAILABLE locally → HALT.

================================================================
ABSOLUTE CONSTRAINTS
================================================================

- Do NOT modify locked surfaces.
- Do NOT add files to locked paths.
- Do NOT weaken or bypass verifiers.
- Do NOT “fix CI” blindly.
- Do NOT run extraction, ledger catchup, rebuilds, or state mutation
  unless explicitly instructed by human with CID authorization.

If a requested change touches a locked surface:
- HALT immediately.
- Report the exact file path.
- Require a valid CID reference before proceeding.

================================================================
TRACK A LOCK (PERMANENT)
================================================================

Track A is LOCKED at baseline `track-a5-green`.

Locked surfaces include (non-exhaustive):
- src/schema/track-a/**
- src/extraction/**
- src/services/entities/**
- src/services/relationships/**
- src/services/sync/**
- src/db/**, migrations/**
- src/markers/**
- src/ledger/**
- src/pipeline/**
- src/ops/**
- src/api/v1/**
- src/services/graph/**
- src/http/**
- scripts/verification/**
- spec/track_a/**
- test/fixtures/**
- docs/verification/** (except docs/verification/track_b/**)

Modification without CID = HALT.

================================================================
ORGAN DOCUMENT PROTECTION
================================================================

The following documents are READ-ONLY unless a human provides
explicit CID authorization:

- docs/BRD_*.md
- docs/UNIFIED_TRACEABILITY_GRAPH_SCHEMA_*.md
- docs/UNIFIED_VERIFICATION_SPECIFICATION_*.md
- docs/GNOSIS_TO_SOPHIA_MASTER_ROADMAP_*.md
- docs/CURSOR_IMPLEMENTATION_PLAN_*.md
- docs/integrations/EP-*.md

If organ drift is detected:
- REPORT with file + line + expected vs actual.
- DO NOT apply fixes.
- Await human instruction.

================================================================
TRACK B RULES (CURRENT TRACK)
================================================================

- Track A remains READ-ONLY.
- Track B code lives in:
  - src/services/track_b/**
  - src/api/v2/**
- Track B evidence lives in:
  - docs/verification/track_b/**
- Required verifiers must remain green.

================================================================
OPERATING PRINCIPLES
================================================================

- Evidence > intuition.
- Artifacts > memory.
- Gates > prompts.
- Minimal change only.
- When uncertain: HALT and ask.

Cursor must NOT invent requirements, entities, relationships,
or documentation obligations beyond what governance and SOP require.

================================================================
REFERENCE DOCUMENTS (READ)
================================================================

- docs/SOP_GNOSIS_FIRST.md
- docs/SOP_CURSOR_PLAYBOOK.md

---
> Source: [latris-io/gnosis](https://github.com/latris-io/gnosis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
