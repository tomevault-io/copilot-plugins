## structuralowf

> Guidance for AI assistants working on P≠NP via Semantic Conservation Law formalization.

# CLAUDE.md

Guidance for AI assistants working on P≠NP via Semantic Conservation Law formalization.

## Critical Warnings

**NEVER run `lake clean`** — Mathlib rebuild takes 2-4+ hours. Use only `lake build` (incremental).

**NEVER create documentation files** unless explicitly requested.

**NEVER use tables in `paper/read-or-x.md`** — Use lists instead. Tables are allowed in Lean documentation and README files.

**NEVER revert files** (backups, git checkout, git revert) unless explicitly asked. Commit to fixes, don't abandon progress.

---

## Project Overview

**Claim**: P≠NP via information-theoretic approach (Semantic Conservation Law)

**Status**: Publication-ready
- ~80,000 lines Lean 4 (165 files)
- 0 sorries, 2 core axioms
- Full build passes

**Key Resources**:
- `paper/read-or-x.md` — Mathematical exposition
- `lean/` — Mechanized proof (authoritative)
- `docs/CONTRIBUTIONS.md` — Reviewer guidelines
- `docs/AI_REVIEW_GUIDE.md` — AI-assisted review guide

---

## Directory Structure

```
lean/
├── Layer0_Foundations/   — SCL framework
├── Layer1_Construction/  — L* construction
├── Layer2_StructuralOWF/ — One-way function
├── Layer3_InformationBounds/ — Lower bounds
├── Layer4_Operational/   — TM semantics
└── Layer5_Applications/  — P≠NP theorem + Crypto

paper/                    — Paper sources
docs/                     — Documentation
```

---

## Build Commands

```bash
cd lean
lake build              # Incremental build (fast)
lake build LayerName    # Specific layer
```

**FORBIDDEN**: `lake clean` (causes multi-hour rebuild)

---

## Trust Boundary (2 Core Axioms)

1. **`algspec_has_tm`** — Church-Turing bridge (any AlgSpec has TM implementation)
2. **`remaining_indistinguishable`** — WC-1 bridge (indistinguishability axiom; separation and time bound derived)

All standard CS/information-theory principles. See `docs/AXIOM_FINAL_COUNT.md`.

**Note**: Cryptographic applications in `Layer5_Applications/Crypto/` use additional standard axioms (PRG→PRF, OWF→commitment, etc.) that are derived consequences of OWF existence, not part of the core P≠NP proof.

---

## Proof Architecture

```
OWF construction → Information must flow (≥2^Ω(n)) → Flow costs time
→ Poly-time impossible → OWF exists → FP≠FNP → P≠NP
```

**Core insight**: Information-theoretic impossibility (q + Φ ≥ R), not algorithmic analysis.

---

## Key Documentation

- `docs/PROOF_CONTROL_FLOW.md` — 11 critical theorems, proof spine
- `docs/CRITICAL_DEFINITIONS.md` — 108 definitions cataloged
- `docs/AXIOM_FINAL_COUNT.md` — Trust boundary details

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pmarreddy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pmarreddy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
