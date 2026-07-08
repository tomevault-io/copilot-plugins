## elidex

> This repo's engineering rules, architecture, and design philosophy are the

# AGENTS.md

This repo's engineering rules, architecture, and design philosophy are the
**`CLAUDE.md`** at the repo root — read it first; it is the single source of
truth. Do not restate or fork its rules here.

## Review guidelines

Codex reads this section for PR code review (it keys on the literal
`## Review guidelines` heading). The local pre-push gate (Claude-based) already
covers generic correctness and style, so a generic review adds little. Add
value by applying elidex's *project-specific* design lenses and flagging what a
generic reviewer misses:

- Read **`.claude/skills/elidex-review/axes.md`** — the 5-axis review SSoT
  (Layering mandate / ECS-native lens / pragmatic shortcut / spec citation /
  project-context). Apply those axes to the diff.
- Ground design judgments in **`CLAUDE.md`** (§ "Design philosophy",
  § "Layering mandate", § "Spec citation").
- Prioritize design and correctness issues over naming/style nits.
- For spec claims, cite the specific WHATWG / W3C / TC39 / CSS section; do not
  assert from memory.

Those files are the SSoT; this section only directs you to apply them and
deliberately does **not** duplicate their content (duplication would drift —
see CLAUDE.md "One issue, one way").

---
> Source: [send/elidex](https://github.com/send/elidex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
