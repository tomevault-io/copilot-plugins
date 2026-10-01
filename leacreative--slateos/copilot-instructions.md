## ota-lockstep

> OTA hard lock-step — one chunk in flight (N-19); never reopen multi-chunk window


# OTA lock-step (N-19)

Watch AppInbox is **one slot**. Phone OTA must stay hard lock-step.

## Non-negotiable

1. `OtaSenderState.sendable` is **0** whenever `sentOffset != acknowledgedOffset`.
2. Trailing **CREDIT** while a chunk is in flight must be **ignored** (Wait).
3. **ACK** syncs both pointers and opens **exactly one** window (`OTA_WINDOW_BYTES`).
4. Do not “optimise” throughput by advertising >1 chunk of credit against the
   inbox without a firmware inbox-depth change and new tests.

## Before editing

Read `docs/lessons-learned.md` § A (OTA) and `docs/invariant-tests-plan.md` §1.

## Tests

Before packaging DFU or claiming an OTA fix, run:

```powershell
powershell -File scripts/run_invariant_tests.ps1
```

Do not weaken `N19 …` cases in `OtaXferTest`. Add coverage for any new send path.

## Handover

Include: `Do not regress: OTA sendable==0 while sent!=acked (N-19 / p116)`.

---
> Source: [LeaCreative/SlateOS](https://github.com/LeaCreative/SlateOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
