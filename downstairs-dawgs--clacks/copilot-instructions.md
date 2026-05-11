## clacks

> Be concise, precise, and information-dense in your communication.

Be concise, precise, and information-dense in your communication.
No flattery.
No emojis.
Do not claim credit on commit messages and PR descriptions.
Use CLI tools instead of complicated MCP servers etc. whenever possible.
Do not waste my time or your tokens.
Make as much of your work reproducible as you can. Store artifacts (files, etc.) and scripts for how to make use of them. Always make a directory (even a temporary one) for each group of files, unless one has already been specified.
Always respect .gitignore
NEVER amend commits. Always create new commits. No exceptions.
NEVER rebase. Use merge commits to integrate changes.

Before committing, run all checks:
```
uv run ruff check
uv run ruff format --check
uv run mypy src/
uv run python -m unittest discover -s tests
```

---
> Source: [downstairs-dawgs/clacks](https://github.com/downstairs-dawgs/clacks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
