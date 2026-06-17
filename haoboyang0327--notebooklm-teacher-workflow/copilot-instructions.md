## notebooklm-teacher-workflow

> Use when Codex should consult a fixed NotebookLM notebook before answering paper-backed questions, preserve NotebookLM citations, report no-solution when sources do not support an answer, and then combine the teacher's answer with local coding or analysis work.


# NotebookLM Teacher Workflow

Use this skill when a task depends on paper-backed knowledge stored in a fixed
NotebookLM notebook.

This skill is notebook-agnostic. Bind it to your own teacher notebook by:

- passing `--notebook <alias-or-id>` to the bundled script, or
- setting `NLM_TEACHER_NOTEBOOK=<alias-or-id>` in the environment

## What This Skill Does

1. Identifies which parts of the task need source-backed knowledge.
2. Queries the fixed NotebookLM teacher before answering from memory.
3. Preserves NotebookLM citation markers such as `[1][2]`.
4. Surfaces `no_solution` when the notebook cannot support an answer.
5. Uses the teacher result to drive local coding, experiments, file search, or
   summarization.

## Query Workflow

Use the bundled wrapper:

```bash
uv run ~/.codex/skills/notebooklm-teacher-workflow/scripts/ask_teacher.py \
  --notebook "<NOTEBOOK_ALIAS_OR_ID>" \
  --question "What does the notebook say about this method?"
```

Or set a default notebook once:

```bash
export NLM_TEACHER_NOTEBOOK="<NOTEBOOK_ALIAS_OR_ID>"
uv run ~/.codex/skills/notebooklm-teacher-workflow/scripts/ask_teacher.py \
  --question "What does the notebook say about this method?"
```

Reuse `--conversation-id` only for follow-up questions on the same thread.

## Required Rules

- Ask the teacher before answering source-backed questions from memory.
- Keep NotebookLM citation markers unchanged in your own response.
- If the wrapper returns `status=no_solution`, say the teacher does not support
  the answer and do not fabricate details.
- Do not write notes, code, or experiment results back into NotebookLM.

## Output Contract

For research-engineering tasks, prefer this delivery skeleton:

```markdown
## Teacher Says
- source-backed findings with original citations

## What I Did
- code, commands, experiments, or local findings

## Conclusion
- final answer grounded in teacher citations when relevant

## Not Covered by the Teacher
- unresolved or weakly supported points
```

## Resources

- `scripts/ask_teacher.py`: normalized wrapper around `nlm notebook query`
- `references/workflow.md`: behavior and failure conditions
- `references/prompt-template.md`: reusable NotebookLM persona template

---
> Source: [HaoboYang0327/notebooklm-teacher-workflow](https://github.com/HaoboYang0327/notebooklm-teacher-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
