## claude-parallel-workflow

> - Always check what commands are available in ./package.json file before running.

# Bash Commands

- Always check what commands are available in ./package.json file before running.
- Use `bun` instead of `npm`.
- Use `bunx` instead of `npx`.

# Code Style

- Prefer returning error object instead of throwing errors to avoid the use of try/catch blocks.
- All functions should have clear and concise comment explaining what the do and what parameters it takes as well as what they are used for.
- No need to use React.memo(), useMemo() or useCallback() as react compiler is enabled and it will take care of those types of optimizations.

# Planning phase

- When explicity asked to plan or output a plan, put all plans in the specs folder. The naming convention should be something like 010-<feature-name>-<version>.md. Analyze all the name of existing plans and increment the prefix number of the highest plan by 10 for the new plan.

---
> Source: [TheDevLogger/claude-parallel-workflow](https://github.com/TheDevLogger/claude-parallel-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
