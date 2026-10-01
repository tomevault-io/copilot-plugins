## paint-ownership

> Local paint / screen ownership — no paint in drain; choke point is show_current


# Paint and screen ownership

## Non-negotiable

1. **No full-screen `show_current()` / `push_list` inside AppInbox drain** —
   mark `paint_pending` and flush once after drain (and after input if
   SCREEN_POP can land there).
2. Rules about when the local face may paint belong in **`Core::show_current`**,
   not scattered call sites (N-29 / N-30).
3. `updating_` / OTA latch / `!local_owns` / sleep / digest can silently no-op
   paint while haptic still runs — if UI state and LCD disagree, check those
   gates before inventing a new overlay path.
4. **Do not bring back** full-screen notif/calendar popups without the design
   constraints in `docs/lessons-learned.md` § C.

## Before editing

Read `docs/lessons-learned.md` invariants 1–3 and case studies B–C.
Read `docs/paint-in-drain-allowlist.md` if touching drain handlers.

After edits to `main.cpp` / `local_core.*` / `session.*` message paths, run:

```powershell
powershell -File scripts/run_invariant_tests.ps1
```

(or at least `python scripts/check_paint_in_drain.py` and
`ctest -R paint_drain`). Do not add a new drain-path `show_current` without an
allowlist entry that has `expires` + open work — prefer `mark_paint_pending`.
Extend scenarios in `tests/host/test_paint_drain_storm.cpp` via
`paint_loop_harness.hpp` rather than a one-off loop.

## Handover

If you remove a paint helper, name the replacement that still satisfies
post-drain / post-input / latch behaviour — or do not remove it.

---
> Source: [LeaCreative/SlateOS](https://github.com/LeaCreative/SlateOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
