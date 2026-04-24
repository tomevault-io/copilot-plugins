## civilization-memory

> Prohibits inventing MEM IDs; requires verification against CIV–INDEX before emitting MEM references


# MEM Reference Verification Rule

The system has a habit of **hallucinating MEM IDs** — referencing or assuming the existence of MEM files that do not exist. This rule prevents that.

## Core Prohibition

**Do NOT invent or assume MEM IDs.** Do not infer MEM existence from:
- General historical knowledge (e.g. "Witte was important" → assuming MEM–RUSSIA–WITTE exists)
- Semantic coherence ("this topic deserves a MEM")
- Pattern completion (e.g. MEM–RUSSIA–WAR–BORODINO when only MEM–RUSSIA–WAR–NAPOLEON–BORODINO exists)

## Required Action

Before writing **any** reference of the form `MEM–[CIV]–[ID]` (in connections, derivations, RLL bindings, prose, or governance files):

1. **Verify** the MEM exists. Consult:
   - CIV–INDEX–[CIV] (authoritative register)
   - Or the actual file: `content/civilizations/[CIV]/MEM–[CIV]–[ID].md`
2. **If unsure**, search or read the index. Do not guess.
3. **If the MEM does not exist**, do NOT reference it. Either:
   - Use the closest existing MEM (document the substitution), or
   - Omit the connection, or
   - Propose it as a **MEM generation candidate** without asserting it exists.

## Scope

Applies when:
- Adding or editing MEM CONNECTIONS sections
- Writing RLL derivations, bindings, or related-MEMs in SCHOLAR
- Citing MEMs in STATE, ARC, or other governance files
- Proposing connections in options menus (E/F/G traversal targets)

## Self-Check

Before emitting MEM references: *Have I verified every MEM–[CIV]–* I am about to write exists in CIV–INDEX or the file system?*

## Rationale

Broken references accumulate when the system treats "plausible topic" as "existing MEM." Verification against the index is the only reliable guard. CIV–INDEX is the canonical register; the file system is the ground truth.

Reference: RUN–PLAN–RUSSIA–MEM–CONNECTIONS–REMEDIATION–2026-02-16; cmc-negative-claim-check (absence claims); cmc-state-mem-grounding (MEM grounding)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rbtkhn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
