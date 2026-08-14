## sessions-flow-backend

> <!-- lean-ctx-rules -->

<!-- lean-ctx-rules -->
<!-- version: 1 -->

# lean-ctx

MANDATORY MAPPING:
• Read/cat -> ctx_read(path, mode)
• Glob/find -> ctx_glob(pattern)
• Shell/bash -> ctx_shell(command)
• Grep -> ctx_search(pattern, path)
• ls/find -> ctx_tree(path, depth)

NEVER use native Read/Grep/Shell/Glob when ctx_* equivalents are available.
Full rules: ~/.cursor/rules/lean-ctx.mdc (auto-loaded) — do not duplicate here.
<!-- /lean-ctx-rules -->

---
> Source: [salah-sa/Sessions_Flow-backend](https://github.com/salah-sa/Sessions_Flow-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
