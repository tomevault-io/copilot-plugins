## velo-oracle-prime

> > **SINGLE SOURCE OF TRUTH: `docs/current/ONE_TRUTH.md`**

# VÉLØ PRIME — Claude Code Context

> **SINGLE SOURCE OF TRUTH: `docs/current/ONE_TRUTH.md`**
> That file wins any conflict with this file or any other doc.

## Quick orientation

- **Operational law:** `docs/current/ONE_TRUTH.md`
- **Step-by-step daily commands (Steps 1–20):** `THE_ONE_TRUTH.md` (root)
- **Race day lifecycle:** `docs/current/RACE_DAY_RUNBOOK.md`
- **Learning gate:** `docs/current/LEARNING_ADMISSION_GATE.md`
- **VFU state:** `docs/current/VELO_VFU_TIMELINE_APPENDIX.md`
- **VCP (coherence protocol):** see VCP State section in ONE_TRUTH.md

## Hard laws (never override without operator approval)

1. Racing API is PERMANENTLY DECOMMISSIONED for live use. RP HTML is the only live source.
2. No live staking. Execution bridge is SIM/PAPER only with hard runtime guards.
3. Live model weights are FROZEN — no promotion without operator gate.
4. RPDC is horse-career memory. PDF intelligence must never overwrite RPDC fields.
5. Mission Control derives source truth from the observability packet — never by default. Missing = UNKNOWN.
6. Sigma Telegram format is LOCKED — never change it. Always use `run_results_sigma.py`.
7. No new numbered truth files. ONE_TRUTH.md is the only living truth.

## Session start

```bash
PYTHONPATH=. venv/bin/python scripts/ops/velo_session_start_check.py
```

Then follow Steps 1–20 in `THE_ONE_TRUTH.md`.

---
> Source: [elpresidentepiff/velo-oracle-prime](https://github.com/elpresidentepiff/velo-oracle-prime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
