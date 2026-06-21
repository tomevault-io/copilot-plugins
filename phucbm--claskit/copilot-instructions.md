## claskit

> >


# claskit

## Your job

From current conversation context (plan, feature, brainstorm), generate `.claude/tasks/todo/<slug>.md` in user's working directory.

## Rules

- Derive `<slug>` from task title: lowercase, spaces → hyphens, strip special chars
- Output path: `.claude/tasks/todo/<slug>.md`
- Create folder if not exist: `mkdir -p .claude/tasks/todo`
- Write immediately — no confirmation needed
- After writing confirm: `Created .claude/tasks/todo/<slug>.md`

## File format

```markdown
# <Title>

## Task

<What needs to be built. 2-5 sentences from context.>

## Acceptance Criteria

- [ ] <criterion>
- [ ] <criterion>

## Files Affected

| File | Change |
|------|--------|
| `<path>` | <description> |

## Notes

<Dependencies, order hints, constraints. Omit section if nothing to add.>
```

## Guidelines

- Acceptance criteria: concrete, testable, checkbox format
- Files Affected: best guess from context; omit rows if unknown
- Notes: only include if context has dependency or constraint info
- Task section: factual, no fluff

---
> Source: [phucbm/claskit](https://github.com/phucbm/claskit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
