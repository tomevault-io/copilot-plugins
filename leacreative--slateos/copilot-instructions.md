## ambient-power

> Ambient profile is radio-only while awake — power::Ambient only when Core sleeping (m44)


# Ambient vs display sleep (m44)

## Non-negotiable

1. `apply_profile(Ambient)` = **radio interval only** — must not blank LCD.
2. `power::enter(Ambient)` turns backlight off + ST7789 SLPIN. Call it **only
   when `Core` is sleeping** (or from Core’s own sleep path with matching wake).
3. If Core is awake and `power::current() == Ambient`, enter **Active** (or
   equivalent lit state). `wake_seconds == 0` does **not** protect against
   Ambient blanking.
4. Symptom “face blank, swipes work” after Ready/OTA → check Ambient/power
   **and** ownership/paint (differential, not last milestone only).

## Before editing

Read `docs/lessons-learned.md` invariant 4 and case study B; see
`docs/invariant-tests-plan.md` §2.

Before packaging DFU or claiming an Ambient/blank-face fix:

```powershell
powershell -File scripts/run_invariant_tests.ps1
```

## Handover

Include: `Do not regress: power::Ambient only while Core sleeping (m44)`.

---
> Source: [LeaCreative/SlateOS](https://github.com/LeaCreative/SlateOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
