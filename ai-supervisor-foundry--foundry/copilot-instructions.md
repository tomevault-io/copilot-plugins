## contexts

> Instructions for using context files


## Project Context Files (`contexts/`)

1. Context files in `contexts/` provide detailed project documentation for agents with zero context.
2. Read relevant context file before working on a project (e.g., `contexts/easeclassifieds/frontend.md` for FE work).
3. Context files contain project structure, recent fixes, configuration, and common issues.
4. Update context files when making significant changes to keep them current.
5. Use context files to quickly onboard new agents or understand project state.

## Supervisor Context Files (`supervisor-contexts/`)

1. **Index**: Read `supervisor-contexts/CONTEXT.md` first (<100 lines). It links to section files (overview, architecture, validation, usage, etc.).
2. **Section files**: Read specific files as needed (e.g., `validation.md`, `tool-contracts.md`).
3. **Sliding Window**: Check `supervisor-contexts/windows/` for recent changes.
4. **Usage**: Use when working on the supervisor system itself (not project tasks).
5. **Updates**: Window files are manually maintained—do not auto-generate.

---
> Source: [ai-supervisor-foundry/foundry](https://github.com/ai-supervisor-foundry/foundry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
