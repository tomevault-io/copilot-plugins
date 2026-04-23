## ghfs

> - CLI related logics, usage of `@clack/prompts`, `console.log` can only be under `src/cli/`

# Rules

- CLI related logics, usage of `@clack/prompts`, `console.log` can only be under `src/cli/`
- GitHub related logics, usage of `octokit` can only be under `src/providers/github/`
- Pure/simple functions should better be under `src/utils/` and be tested alongside.
- Avoid duplicating logics, refactor them to reuse.
- Always use `pathe` instead of `node:path`
- At this moment, we don't care about breaking changes at all, don't worry about migration or backward compatibility.

---
> Source: [antfu/ghfs](https://github.com/antfu/ghfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
