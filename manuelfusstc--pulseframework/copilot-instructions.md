## pulse

> PULSE Framework - Call pulse_status on EVERY message


# 🛑 PULSE: Call MCP tools!

## EVERY MESSAGE → pulse_status FIRST

```
CallMcpTool: pulse_status
Arguments: { "userMessage": "<user message>" }
```

## AFTER CODE CHANGES → pulse_doctor

## EVERY 10 MIN → pulse_checkpoint

| Trigger | Tool |
|---------|------|
| Every message | `pulse_status` |
| After edits | `pulse_doctor` |
| 10+ min work | `pulse_checkpoint` |
| "wrong"/"stop" | `pulse_correct` |
| 2-3 failures | `pulse_escalate` |
| Task done | `pulse_review` |

## NON-NEGOTIABLE

- ⏱️ MAX 30 min autonomous
- 🗑️ NO delete without confirmation
- 📤 NO push without confirmation
- 🔐 NO secrets in code

---

**DID YOU CALL pulse_status? If not, do it NOW.**

---
> Source: [manuelfussTC/PulseFramework](https://github.com/manuelfussTC/PulseFramework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
