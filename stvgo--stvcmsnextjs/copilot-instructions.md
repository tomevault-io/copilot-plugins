## stvcmsnextjs

> Always create **separate commits for each logical change**. Never bundle unrelated changes into a single commit.

# Project Rules

## Git Workflow: Atomic Commits (MANDATORY)

Always create **separate commits for each logical change**. Never bundle unrelated changes into a single commit.

### Rules
1. **One change = one commit**. Each commit must have a single clear purpose.
2. **Never mix functional changes with cosmetic/formatting changes** in the same commit.
3. **Separate by concern**: Multiple features, bug fixes, or refactors → separate commits for each.

### Commit message format
Use conventional commits: `type: description`
- `feat:` new feature
- `fix:` bug fix
- `chore:` maintenance tasks
- `refactor:` code restructuring (no behavior change)
- `style:` formatting changes (no behavior change)
- `docs:` documentation only

### Before pushing
If multiple logical changes were made, ensure they are in **separate commits**. If files for different changes were staged together, commit them in separate groups.

---
> Source: [stvgo/stvCmsNextJS](https://github.com/stvgo/stvCmsNextJS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
