## token-router

> Use `token-router` to avoid loading oversized evidence or long routeable instruction references into Claude Code when the task only needs a focused slice.

# token-router Claude Code Bootstrap

Use `token-router` to avoid loading oversized evidence or long routeable instruction references into Claude Code when the task only needs a focused slice.

## Router Commands

For logs and stack traces:

```bash
OLLAMA_NUM_CTX=4096 OLLAMA_KEEP_ALIVE=0s \
python3 scripts/router.py error_log path/to/file.log --query "error or incident terms"
```

For long source files:

```bash
OLLAMA_NUM_CTX=4096 OLLAMA_KEEP_ALIVE=0s \
python3 scripts/router.py heavy_code path/to/file.py --query "bug or code-path terms"
```

For long routeable agent-context references such as detailed `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.cursorrules`, or `agent-context/*.md` files:

```bash
OLLAMA_NUM_CTX=4096 OLLAMA_KEEP_ALIVE=0s \
python3 scripts/router.py agent_context path/to/agent-context.md --query "task-specific rule terms"
```

## Operating Rules

- Treat the local model as a line router, not as a summarizer.
- Analyze only the raw slices returned by the router.
- If a slice is too narrow, request or inspect wider nearby line ranges.
- Keep mandatory always-on Claude rules short. The router cannot reduce the token cost of a long `CLAUDE.md` after Claude Code has already loaded it.
- Put long task-specific Claude Code guidance in routeable reference files, then use `agent_context` when those details matter.

---
> Source: [sleeplesshan/token-router](https://github.com/sleeplesshan/token-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
