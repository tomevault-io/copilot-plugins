## f-r-i-d-a-y

> Global agent speech kill switch — when FRIDAY_CURSOR_NARRATION is false, Composer must never call friday-speak or pick-session-voice (overrides all other Friday narration rules)


# Cursor agent narration — global rule (overrides other voice rules)

This file is **always applied**. It is the **single global switch** for whether the Cursor Composer agent may use voice (`friday-speak.py`, `pick-session-voice.py`) for its own narration.

**Check every turn:** read `FRIDAY_CURSOR_NARRATION` from `d:\code\openclaw\.env`.

If the value is **false**, **0**, **off**, or **no** (case-insensitive):

- **Do not** run `pick-session-voice.py` for this chat.
- **Do not** run `friday-speak.py` for acknowledgements, mid-task status, completion summaries, WIP checkpoint announcements, questions, thinking narration, subagent-style speak, or any other agent-initiated TTS — **text replies only. No exceptions** unless the user explicitly asks you to speak a line in this turn.
- Ignore the mandatory narration instructions in `friday-narrate.mdc`, `acknowledge-before-planning.mdc`, `narrate-thinking.mdc`, `friday-thinking-sounds.mdc`, and spoken steps in `completion-read-memory.mdc` / `wip-safety-commit.mdc` for those cases.

**Full Cursor-side silence (no JSONL echo either):** with narration off, also set `FRIDAY_CURSOR_SPEAK_REPLY=false`, `FRIDAY_CURSOR_SPEAK_SUBAGENT_REPLY=false`, and `FRIDAY_CURSOR_SPEAK_THINKING=false` in `.env` so `cursor-reply-watch` does not speak transcript or thinking lines. Daemons and pc-agent task-complete TTS are separate.

If the variable is **missing**, or is **true**, **1**, **on**, or **yes** (case-insensitive), follow the other Friday narration rules as written.

**Unchanged when narration is off:** voice listen/ambient daemons, Alexa, WhatsApp, skill-gateway or pc-agent task-complete TTS, and when the **user** explicitly asks you to speak something.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theroyalraj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/theroyalraj)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
