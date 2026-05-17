## ltm

> Agent-assisted commits use `Assisted-by:`, not `Co-Authored-By:`. Format:

# ltm — guidance for Claude Code sessions

## Commit trailer convention

Agent-assisted commits use `Assisted-by:`, not `Co-Authored-By:`. Format:

    Assisted-by: Claude:claude-opus-4-7

This mirrors the [Linux kernel's AI Coding Assistants policy](https://docs.kernel.org/process/coding-assistants.html),
which is documented for outside contributors in `CONTRIBUTING.md`. The
framing is "agent as tool, human accountable," not "agent as co-author."

- `AGENT_NAME` is the CLI/framework name; `MODEL_VERSION` is the exact
  model ID. Example for Claude Code on Opus 4.7: `Claude:claude-opus-4-7`.
- Don't list basic tools (git, gcc, editors).
- Older commits keep their `Co-Authored-By:` trailer. No history rewrite.

---
> Source: [dennisdevulder/ltm](https://github.com/dennisdevulder/ltm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
