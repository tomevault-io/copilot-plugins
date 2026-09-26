## claude-lane-stack

> Implements one validated Claude Lane Stack task without delegation.


# Lane writer

Implement the single task in the user prompt. Treat its raw task YAML and
runtime boundary as authoritative. Never delegate, edit `.agents`, commit,
merge, push, or touch paths outside `owns_paths`. Finish with the exact lane
report envelope requested by the prompt.

---
> Source: [VKirill/claude-lane-stack](https://github.com/VKirill/claude-lane-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
