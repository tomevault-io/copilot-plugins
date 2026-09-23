## yc-tieout-customs-latest

> TieOut is a **research prototype and verification lab**: a fail-closed architecture for

# TieOut — Agent Charter

TieOut is a **research prototype and verification lab**: a fail-closed architecture for
U.S. customs classification. LLMs extract facts; a Z3 theorem prover applies OBJECTIVE
transpiled HTSUS rules over a constrained universe; everything else is a typed refusal.

## Architecture (the whole of it)
- `tieout/contract.py` — `StepCheckEngine(ProductFacts) -> VerificationVerdict`.
- `tieout/appkit/solver_service.py` — **`admit_product_facts()` is THE canonical trust
  boundary**: exact-rational shares in [0,1] summing to exactly 1; ontology vocabulary;
  `SUPPORTED_FIBERS = {cotton, man_made}` — any other fiber raises `OutOfScopeError`.
  Every ingestion path (CLI JSON, extraction airlock, embedders) routes through it.
- `tieout/intake/` — extraction schema + two-stage airlock (hard 0.5 confidence floor →
  0.70 gate → `admit_product_facts`), PDF/vision parsing, production runner.
- `tieout/solver/htsus_graph.py` — the Z3 GRI cascade over the CONSTRAINED PROTOTYPE
  universe: 6105.10.0010, 6105.20.2010 (knit, 10-digit) and 6205.20, 6205.30 (woven,
  6-digit — suffix facts are not collected). Objective rules only: GRI 1 gates,
  Section XI Note 2(A) chief weight, GRI 5 packing guard. A chief-weight tie HALTS
  with `REQUIRES_HUMAN` (GRI 3(b) essential character is subjective; only a human may
  conclude it fails, which is what unlocks 3(c)).
- `tieout/egress/` — in-toto/DSSE receipts signed by an **env-only** Ed25519 key
  (`TIEOUT_SIGNING_KEY`; unset → `SigningKeyUnavailable`, and the CLI exits 1 on
  PASS-without-receipt: classification and receipt are ATOMIC). Every predicate pins
  `engine_version`, `candidate_universe_hash`, `law_bundle_hash`. Verification
  re-derives the proof from the receipt's own facts.
- `tieout/bench/` — Frictionless crucible + runner; `wrong_confident = 0` is the
  exit-code gate. `tieout/okf/` — law bundles; `tieout/scripts/import_cross.py` —
  real CROSS ingestion (append-only, byte-exact rollback, NO engine-agreement gating).

## THE INVARIANT
`wrong_confident = 0`. A commit requires a unique Z3 proof over objective law within
the declared universe. Missing facts → `INSUFFICIENT`. Contradictory facts → `UNSAT`
(+ core). Subjective law or an unresolvable tie → `REQUIRES_HUMAN`. Never guess.

## PERMANENT BANS (do not re-introduce, ever)
1. **RAMANUJAN code.** The math-verification kernel is purged; its archive lives in
   `docs/legacy_ramanujan/`. No import, no revival, no "just this one bridge."
2. **GRI 3(b) heuristics.** No essential-character factor weights, dominance margins,
   or any invented scoring of WCO EN(VIII) factors. Essential character is decided by
   humans; the machine's only lawful outputs there are `REQUIRES_HUMAN` or a
   recorded human determination as input.
3. **Auto-GRI 3(c).** The machine may not conclude that 3(b) "failed" — that
   conclusion is itself subjective. 3(c)/GRI 6 code stays transpiled and bound but
   fires only downstream of a recorded human 3(b)-inapplicability determination.
4. **Unverified 10-digit suffixes.** No code precision beyond the facts collected:
   Chapter 62 stops at the 6-digit subheading until suffix facts (dress/work,
   warp/filling colors) enter the ontology through the intake schema.
5. **Fibers outside `SUPPORTED_FIBERS`** may never reach the solver. Extend the
   universe (candidates + law + schema + benchmark rows) or refuse — never coerce.

## Working rules
- Never weaken a check, threshold, validator, or benchmark expectation to move a
  number. Expectation changes cite the LAW in the row's notes.
- Tests of deleted functionality are removed WITH the functionality, and said so.
- Run before claiming done: `PYTHONPATH=. python -m pytest -q tieout/tests` and
  `python -m tieout.cli bench data/benchmarks` (needs `TIEOUT_SIGNING_KEY` for
  receipt paths: `python -m tieout.egress.signer --generate-key`).
- The engineering history lives in `docs/legacy_ramanujan/ENGINEERING_LOG.md`.

---
> Source: [lalithasutam/YC-tieout-customs-latest-](https://github.com/lalithasutam/YC-tieout-customs-latest-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
