## agentic-developer-handbook

> Project principles live in [.cursor/rules/project-principles.mdc](.cursor/rules/project-principles.mdc), [CONTRIBUTING.md](CONTRIBUTING.md), and [docs/adr/](docs/adr/README.md). Follow them.

# Claude Code instructions

Project principles live in [.cursor/rules/project-principles.mdc](.cursor/rules/project-principles.mdc), [CONTRIBUTING.md](CONTRIBUTING.md), and [docs/adr/](docs/adr/README.md). Follow them.

## Commit authorship

- Commit with the repository owner's configured Git identity only.
- Never add Claude, Anthropic, Cursor, or any other AI tool as a commit author or co-author.
- Never add AI `Co-Authored-By` trailers to commit messages, and no "Generated with" AI attribution lines in commits or pull request descriptions.
- Never change `git config user.name` or `user.email` automatically.

## Validate before a pull request

```sh
./mvnw verify
python -m unittest discover -s scripts/tests -v
```

Neither needs an API key or network model calls. The canonical Java version is 27 (ADR 0005); do not use preview or incubator features.

---
> Source: [cagridursun/agentic-developer-handbook](https://github.com/cagridursun/agentic-developer-handbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
