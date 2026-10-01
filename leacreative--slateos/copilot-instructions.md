## lessons-learned

> Mandatory regression gates for paint/OTA/BLE/power — read lessons-learned, checklist, Do not regress


# Regression gates (mandatory)

You **will** follow these. Recurring blank-face / OTA / Ambient bugs cost real
operator time; prose alone was not enough.

## Before coding (gated areas)

If the task touches **any** of: paint / display lists, OTA, BLE session or
reconnect, power / Ambient, screen ownership, notifications overlays:

1. **Read** `docs/lessons-learned.md` (or confirm you already did this session).
2. **Skim** the matching section (OTA, Ambient, ownership, popups).
3. Do **not** reintroduce items under **Do not bring back** (full-screen
   notif/calendar popups without a proven design).

## Before claiming done / packaging DFU

Run the anti-pattern checklist in `docs/lessons-learned.md`. In particular:

- No full-screen paint inside AppInbox drain
- OTA: `sendable == 0` while `sentOffset != acknowledgedOffset`
- `power::enter(Ambient)` only when Core is sleeping
- No call-site-only paint gates that belong in `Core::show_current`

**If the change touches OTA or Ambient/power display policy**, also run in the
same turn (must exit 0):

```powershell
powershell -File scripts/run_invariant_tests.ps1
```

See `.cursor/rules/invariant-tests.mdc`.

## Required in the user-facing handover

End with:

```text
Do not regress:
- …
- …
```

If you removed a helper or “cleaned up” a gate, name the replacement choke
point. Silent deletion of m40-style paint helpers / Ambient gates is a
regression.

## Same-turn docs

- Update `docs/issue-prompts-open.md` current state for the fix.
- If the fix teaches a durable rule, add it to `docs/lessons-learned.md`.

Full policy: `AGENTS.md`, `docs/agent-enforcement.md`.

---
> Source: [LeaCreative/SlateOS](https://github.com/LeaCreative/SlateOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
