## nabla-site-python

> Pre-commit hooks configuration and usage guide


# GitHooks & Pre-Commit

This project makes use of `pre-commit` for quality assurance. If pre-commit changes any files, it is okay to re-add and re-commit. If there are any errors, you should fix them before doing so.

## Useful Commands

```bash
# Test hooks without committing
pre-commit run --all-files

# Commit without running a hook
SKIP=<hook-id> git commit
```

## Hook Categories

The pre-commit configuration includes several categories of hooks:

- Security Tools
- Code Quality Tools
- Filesystem tools
- Git quality
- Tests

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AlbanAndrieu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
