## harness-lessons

> Harness-learned lessons (auto-synced from gate failures)


# Harness lessons

Auto-synced from gate failures; do not hand-edit.

Learned harness lessons (auto-synced; do not hand-edit):
- [stagnation/active/core] Identical validation fingerprint repeated. Change approach — do not re-apply the same failing edit.
  avoid: Do not retry the exact same patch, command, or suppression.
  prefer: Diagnose root cause with a different path, or escalate with BLOCKED / TRIED / NEED.
  before retrying: Diff your last edit against the gate output; ensure the next action is different.
_(5 more active lessons omitted under char budget)_

---
> Source: [gugamistri/gflow](https://github.com/gugamistri/gflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
