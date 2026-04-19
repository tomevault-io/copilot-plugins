## aimoneyweb-autopilot-phase36-38

> - Project goal: Fully autonomous, self-healing trading research + execution mesh.

# NeoLight Guardrails (Cursor)
- Project goal: Fully autonomous, self-healing trading research + execution mesh.
- Critical files:
  - phases/*  : phase scripts
  - agents/*  : brain/orchestrator/watchers
  - trader/*  : SmartTrader loop and broker interfaces
  - dashboard/*: live FastAPI dashboard
  - scripts/* : guardians, schedulers, auto-heal
- Coding standards:
  - Python 3.12+, no type|None (use Optional[T]).
  - Fail fast, log errors to logs/*.log.
  - Never break idempotency: scripts must be re-runnable.
- Commit intent:
  - Small PR-sized changes; keep diffs readable.
- Tests (lightweight):
  - At minimum, `python -m py_compile` passes for changed Python files.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tradeplatformnoni-tech) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
