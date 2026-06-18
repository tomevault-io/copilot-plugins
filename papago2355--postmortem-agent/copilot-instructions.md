## postmortem-agent

> Use after a tool retry, user correction, rollback, or 'I was wrong' admission. Captures the failure as a typed postmortem entry, applies the prompt-→param-→info-→code investigation order, and promotes recurring patterns into MEMORY.md.


# Postmortem-Evolve

## Overview

Hermes' built-in learning loop is success-biased: skills are auto-generated from "repeated patterns and successful interactions." Failures don't get the same treatment — they land as freeform diary lines in `MEMORY.md` (or are forgotten). This skill closes the loop by capturing failures with the same structural rigor that successful skills get.

**Core principle**: When you have been confidently wrong, the entry you write next is worth more than ten entries about successes. But only if it's structured to drive a prompt/param/info fix — not a venting note.

## When to Invoke

Invoke this skill at any of these signals during or after a session:

- **Tool retry** — you called a tool, got an error or empty result, then called the same tool with modified args.
- **User correction** — the user said any of: `"no, that's wrong"`, `"actually..."`, `"undo"`, `"that's not right"`, `"you should have..."`, in any language.
- **Rollback** — the user (or you) ran `git reset`, `git checkout --`, restored a file, reverted an edit, or stopped a process you started.
- **Self-admission** — you said `"I was wrong"`, `"let me check"`, `"I assumed"`, `"sorry, that was incorrect"`.
- **Confident failure** — you produced a high-confidence answer that the user disputed within 2 turns.
- **Session-end reflection** — at the end of any non-trivial session (3+ tool calls), pass the transcript through `scripts/detect.py` and review candidates.

If none of these fired, you don't need this skill.

## The Iron Law

```
NEVER write a postmortem entry whose resolution is "the model is bad at X".
```

Every entry's `resolution` field must be one of `prompt-fix`, `param-fix`, `info-fix`, or `code-fix`, **in that order of preference**. If you can't articulate which one, you haven't understood the failure — go back to the investigation.

## The Investigation Order (the part that earns the entry)

Before writing the postmortem, run this four-step check:

1. **Missing tool param?** Could a new parameter (or value) on an existing tool let the agent express what was needed? *Example: a search needed a `title_contains` filter that didn't exist; agent invented an `if "고형제" in query` regex branch instead.*
2. **Missing prompt instruction?** Does the agent's system prompt or skill description tell it this case exists? One sentence with one few-shot example beats thirty lines of code.
3. **Missing observation info?** Does the LLM **see** what it needs to make the call — counts, distributions, schema facts, prior-turn context, error messages from the tool?
4. **Only after 1–3 are exhausted: code.** And when you do write code, prefer a new small parameter over a fixed if/else inside an existing function.

If your fix is in 1–3, the postmortem entry **does** evolve memory — it gets converted into a prompt edit, a tool-schema description tweak, or an observation-shape change. If your fix is "code", the entry still pays for itself: future you knows exactly where the agent was previously confidently wrong.

## Entry Schema

Append entries to `~/.hermes/memories/POSTMORTEMS.md` (separate from `MEMORY.md` to avoid burning the 2200-char system-prompt budget). Use this exact shape — `scripts/detect.py` and `scripts/promote.py` parse it:

```
§
pattern: <≤80-char title; lowercase-hyphen ok>
kind: env-config | api-drift | silent-crash | fixed-pipeline | context-overflow | unsafe-shortcut | hallucinated-api | other
confidence_failure_mode: <one sentence: WHY you were sure you were right>
canonical_correct_path: <one sentence: the verified-correct action>
resolution: prompt-fix | param-fix | info-fix | code-fix
first_seen: <YYYY-MM-DD>
last_seen: <YYYY-MM-DD>
recurrence_count: <int, starts at 1>
sessions: [<session-id>, ...]
```

Use the `memory` tool's `add` action against `POSTMORTEMS.md` (target string), or `write_file` directly.

## Recurrence Escalation

When a pattern's `recurrence_count` reaches **3**, promote it to `MEMORY.md` (the auto-injected store) so the system prompt itself starts pushing back on the failure mode. Promotion is one-way: the postmortem entry stays in `POSTMORTEMS.md` (full corpus), and a terse rule lands in `MEMORY.md`.

Promoted-rule format in `MEMORY.md`:
```
[PROMOTED-FROM-POSTMORTEM] <pattern>: <canonical_correct_path>. (seen Nx, last YYYY-MM-DD)
```

Run `python scripts/promote.py` at session-end (the `hooks/post_session.sh` hook does this automatically; see README). Promotion respects `MEMORY.md`'s 2200-char limit by removing the oldest non-promoted entries first.

## Pruning

Entries with `resolution ∈ {prompt-fix, param-fix, code-fix}` AND no recurrence in 30 days get archived to `POSTMORTEMS.archive.md`. `info-fix` entries are kept longer (90 days) — they're the ones the agent re-reads as a "lessons" corpus.

## What This Skill Is NOT

- **Not** a substitute for `systematic-debugging` — that skill is for debugging *user code*; this skill is for debugging *the agent's own behavior*.
- **Not** a venting log — entries that don't route to a prompt/param/info/code fix are rejected by `scripts/detect.py --strict`.
- **Not** a replacement for `MEMORY.md` — postmortems live separately, only promoted rules cross the bridge.

## Quick Reference

| Step | Action | Tool |
|---|---|---|
| 1. Detect failure signal | Match the trigger list above | manual / `scripts/detect.py` |
| 2. Run investigation | prompt → param → info → code | reasoning |
| 3. Write entry | Append to `POSTMORTEMS.md` with full schema | `memory` action=add or `write_file` |
| 4. Check recurrence | Search prior entries by `pattern + kind` | `scripts/detect.py --check-recurrence` |
| 5. Promote if ≥ 3 | Append terse `[PROMOTED-FROM-POSTMORTEM]` line to `MEMORY.md` | `scripts/promote.py` or `memory` action=add |
| 6. Prune (background) | Archive resolved old entries | `scripts/promote.py --prune` |

See `references/investigation-order.md`, `references/schema.md`, `references/failure-detection.md` for full detail.

---
> Source: [papago2355/postmortem-agent](https://github.com/papago2355/postmortem-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
