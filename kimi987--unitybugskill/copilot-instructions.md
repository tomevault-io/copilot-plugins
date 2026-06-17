## unitybugskill

> Use when the user asks Codex to record, remember, or learn from a bug; when a bug is fixed and its cause should be captured; or when starting related coding/debugging work where prior bug records may prevent repeated mistakes.


# Remember Bugs

## Overview

Maintain a compact, reusable bug memory. Convert each bug into a prevention rule that future coding can check before editing, testing, or declaring work complete.

## Workflow

1. Before related implementation or debugging, read `references/bug-log.md`.
2. Match entries by project, language, framework, file pattern, symptom, or subsystem.
3. Apply every relevant prevention rule while planning, editing, and verifying.
4. When a new bug is found or fixed, append a concise entry to `references/bug-log.md`.
5. If an old entry prevented or explained the issue, update that entry's `Seen again` count or notes instead of duplicating it.

## Recording Format

Append new entries under `## Entries` using this template:

```markdown
### YYYY-MM-DD - Short Bug Title

- Context: Project/module/language/framework.
- Symptom: What failed, including exact error text when useful.
- Trigger: The condition or change that exposed it.
- Root cause: The smallest confirmed cause, not the first guess.
- Fix: The change that resolved it.
- Prevention rule: A concrete check future Codex must run before similar work.
- Verification: Command, test, or manual check that proved the fix.
- Tags: `unity`, `csharp`, `shader`, `editor`, `runtime`, etc.
- Seen again: 0
```

## Prevention Rules

- Prefer confirmed root cause over narrative history.
- Keep each entry specific enough to act on in under one minute.
- Record exact command output or error text only when it improves searchability.
- Add a prevention rule that changes future behavior, such as "check asmdef references before using Odin editor types."
- Before final answers on related work, mention any bug-log rule that affected the solution.

## Common Mistakes

| Mistake | Correction |
| --- | --- |
| Recording "fixed a bug" without root cause | Identify the smallest confirmed cause before logging. |
| Making entries project-specific but unsearchable | Add tags, file patterns, and exact symptoms. |
| Writing broad advice | Convert it into a concrete preflight or verification check. |
| Duplicating repeated bugs | Update the existing entry and increment `Seen again`. |

---
> Source: [kimi987/UnityBugSkill](https://github.com/kimi987/UnityBugSkill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
