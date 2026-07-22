## skill-based-architecture

> Formal docs live under `skills/`. Read `skills/*/SKILL.md` — default to `primary: true` skill; only switch when task clearly matches another skill's description.

# AGENTS.md

{{SUMMARY}}

Formal docs live under `skills/`. Read `skills/*/SKILL.md` — default to `primary: true` skill; only switch when task clearly matches another skill's description.

Conflicts between loaded project instructions → formal docs in `skills/{{NAME}}/` win. This does not override harness-native skill name precedence.

<!-- The <always-applicable> and <task-routing> XML tags below are load-bearing.
     Rationale: LLMs parse XML-tag blocks as discrete hard-constraint sections
     more reliably than plain markdown headings, especially after context
     compression. See skill's references/thin-shells.md § XML-Tag Injection. -->

<always-applicable>

**Always Read (every task, in addition to route-specific reads)**

<!-- ALWAYS_READ_START -->
- `skills/{{NAME}}/rules/project-rules.md`
- `skills/{{NAME}}/rules/coding-standards.md`
- `skills/{{NAME}}/rules/agent-behavior.md`
<!-- ALWAYS_READ_END -->

**Route-before-routing check**: if the request contains vague improvement verbs ("refactor / clean up / optimize / make it better / 整理 / 重构 / 优化") **without** a concrete module/file or verifiable outcome → stop and ask for scope. Do not offer partial plans; see `skills/{{NAME}}/protocol-blocks/ambiguous-request-gate.md` if present.

</always-applicable>

Route metadata lives in `skills/{{NAME}}/routing.yaml`; the bootstrap below tells agents how to match it.

<task-routing>

**Quick Routing (survives context truncation)**

<!-- ROUTING_BOOTSTRAP_START -->
Task routes live in `skills/{{NAME}}/routing.yaml`.

For every new task:
1. Read `skills/{{NAME}}/routing.yaml`.
2. Match by `labels`, `trigger_examples`, and task intent.
3. Read only that route's `required_reads` plus Always Read files.
4. Follow that route's `workflow`.
5. If no route matches, use the `other` route.
<!-- ROUTING_BOOTSTRAP_END -->

</task-routing>

<!-- BEHAVIOR_BLOCK_START -->
## Auto-Triggers

- **New task in same session** → always re-match the route (Common Tasks / `routing.yaml`); the new task may need a different route. Re-read the route's files only if the route changed or context was compacted (a fresh `skills/{{NAME}}/SKILL.md` injection is the signal) — unchanged background stays in context, don't re-read it every task. Can't tell if context compacted? Re-read.
- Before declaring any non-trivial task complete → run Task Closure Protocol (see `skills/{{NAME}}/workflows/task-closure.md`)
- Skip closure only for: formatting-only, comment-only, dependency-version-only, or behavior-preserving refactors
- When user asks to "record/save/remember" something → project-level knowledge goes to `skills/{{NAME}}/` docs; personal preferences go to agent memory

## Red Flags — STOP

- "Just this once I'll skip the AAR" → stop. See `skills/{{NAME}}/workflows/task-closure.md` § Rationalizations to Reject.
<!-- BEHAVIOR_BLOCK_END -->

---
> Source: [WoJiSama/skill-based-architecture](https://github.com/WoJiSama/skill-based-architecture) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
