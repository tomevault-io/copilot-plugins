## completion-traceability

> When completing any task (step, feature, fix, plan), always cite back to the user's original request:


# Completion Traceability

When completing any task (step, feature, fix, plan), always cite back to the user's original request:

1. **Quote or paraphrase** the specific part of the user's prompt this work fulfills
2. **Summarize** what was done to address it
3. **Link files** changed or created

## Why
User writes long prompts and burst prompts across multiple sessions. Without traceability, it's hard to recall what was asked and why something was built.

## Format
```
**Re your request:** "[quote or paraphrase of the specific ask]"
- What was done: [concise summary]
- Files: [list of key files created/modified]
```

## Applies to
- Step completions in multi-step plans
- Final summaries after a task is done
- Session continuations (reference the original session's ask)
- Todo item completions

---
> Source: [HomenShum/nodebench-ai](https://github.com/HomenShum/nodebench-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
