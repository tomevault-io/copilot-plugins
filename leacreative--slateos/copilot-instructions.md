## invariant-tests

> Before OTA/Ambient DFU or claiming those fixes, run scripts/run_invariant_tests.ps1


# Invariant tests (mandatory before OTA / Ambient ship)

The N-19 OTA lock-step and m44 Ambient-display gates are encoded in PC tests.
**Do not package DFU or claim those areas fixed without running them.**

## When you must run

Run from repo root **in the same turn** before:

- Packaging / handing over `slate_dfu` / `slate-dfu.zip` after OTA or Ambient /
  power-policy / `main.cpp` sleep-path changes
- Claiming an OTA stall or blank-face-after-Ready fix is done
- Weakening or “cleaning up” `OtaSenderState`, `ambient_display_action`, or
  the app_loop Ambient switch

Also run after editing any of: `OtaXfer.kt`, `OtaXferTest.kt`,
`SlateOtaService.kt`, `ota_xfer.*`, `ambient_power_policy.hpp`,
`test_ambient_power_policy.cpp`, or the Ambient block in `main.cpp`.

## Command

```powershell
powershell -File scripts/run_invariant_tests.ps1
```

Exit 0 required. Report pass/fail in the handover.

## What it runs

1. `python scripts/check_paint_in_drain.py` (paint-in-drain Layer A)
2. `:sdp-tests:test --tests slate.ota.OtaXferTest` (N-19)
3. Host `test_ambient_power_policy` + `test_paint_drain_guard` +
   `test_paint_drain_storm` (m44 + Layer B + app-loop sim)

Also run after editing `paint_loop_harness.hpp`, `test_paint_drain_storm.cpp`,
or drain-path handlers in `local_core` / `main` / `session`.

Details: `docs/invariant-tests-plan.md`, `docs/paint-in-drain-analysis-plan.md`,
`docs/agent-enforcement.md`.

---
> Source: [LeaCreative/SlateOS](https://github.com/LeaCreative/SlateOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
