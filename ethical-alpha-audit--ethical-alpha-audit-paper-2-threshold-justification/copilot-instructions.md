## prior-portfolio-reproducibility

> Reproducibility and validation discipline


# Reproducibility discipline

This repository follows the EAA reproducibility convention established in Paper 4 (`ethical-alpha-audit-paper-4-historical-replay` is the structural reference):

- `reproduce_all.py` is the single entry point that, given a clean Python environment built from `requirements.lock.txt` (or `environment.lock`), regenerates every file in `outputs/` deterministically.
- `MANIFEST.sha256` records SHA-256 hashes of every file in `outputs/`, `reproducibility/`, and key config files. It is informational, not enforcement.
- `repro_manifest.json` records the human-readable provenance: version, paper, manuscript status, validation entry point.
- `VERSION` is a single line of semver (e.g. `2.1.0`).

When validating reproducibility:

- **Hashes are records, not gates.** Routine commits do not halt on hash differences; the manifest is regenerated to reflect new state. Halting on hash mismatch is reserved for three explicit checkpoints: (1) Stage 8 validation (`validate_v2_1.py`-style); (2) external reproducibility verification (the verifier's hash check against your locked manifest); (3) Zenodo deposit (the deposited archive's hash against the manifest at deposit time).
- **The "circular hash" anti-pattern.** Earlier runbook versions treated hashes as gates, which created loops: the manifest must be regenerated, regeneration changes the hash, halt. v8.5 fixes this at the v8.4 system level (state_fingerprint.ps1 with explicit RECORD/VERIFY modes); paper-repo-level hashes follow the same discipline — record state, do not gate it.
- **Validation gates G-1 through G-11** (Paper 5 convention) are the operative checks. They cover issue-inventory coverage, forbidden-token absence, root-cause execution, patch presence, manuscript-Sobol-N consistency, table population, reproducibility artefact integrity, documentation presence, limitations honesty, and independence-substitution. See `validate_v2_1.py` for the reference implementation.

---
> Source: [ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification](https://github.com/ethical-alpha-audit/ethical-alpha-audit-paper-2-threshold-justification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
