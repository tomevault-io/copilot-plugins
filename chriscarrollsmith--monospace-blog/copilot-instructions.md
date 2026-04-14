## monospace-blog

> Keep commits atomic. Always make the smallest code changes that will achieve your goals.

## Programming philosophy

Keep commits atomic. Always make the smallest code changes that will achieve your goals.

## Resolving Github issues

If asked to resolve a Github issue, check if you're already on an issue branch with `git branch -v`. If not, check if an issue branch already exists with `git branch -r`.

If no issue branch exists, you can create one and switch to it with `gh issue develop $ISSUE_NUMBER -c` (assuming you have set the `ISSUE_NUMBER` variable in your shell).

Make your changes, validate them, then open a pull request like `gh pr create -t "Descriptive title" -b $(cat temp/summary_of_changes.md)` (assuming you have written a `temp/summary_of_changes.md` file).

## Python dependency management with `uv`

This project uses `uv` to manage dependencies and run Python files.

Add Python dependencies to the workspace like:

```bash
uv add pydantic # add --dev flag for development dependencies
```

## Python programming patterns

Always write type-annotated Python code that will pass a `mypy` check.

## HTML templating patterns

Avoid introducing new named CSS classes. This project is themed, and the theme should mostly be controlled by element selectors rather than class or id selectors.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chriscarrollsmith)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chriscarrollsmith)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
