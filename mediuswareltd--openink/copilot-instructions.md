## openink

> This file repeats the rules in [AGENTS.md](../AGENTS.md); keep the two in sync. Full guide: [CONTRIBUTING.md](../CONTRIBUTING.md).

# Instructions for GitHub Copilot

This file repeats the rules in [AGENTS.md](../AGENTS.md); keep the two in sync. Full guide: [CONTRIBUTING.md](../CONTRIBUTING.md).

## Commit messages and pull request titles: Conventional Commits

Use this format, always:

```
<type>(<scope>): <description>

<body: why, not what>

<footers: Closes #12, BREAKING CHANGE: ...>
```

- Types: `feat`, `fix`, `perf`, `docs`, `style`, `refactor`, `test`, `build`, `ci`, `chore`, `revert`.
- Optional scopes: `cli`, `build`, `dev`, `export`, `blocks`, `render`, `runtime`, `spec`, `styles`, `themes`, `examples`, `templates`, `assets`, `deps`.
- Description: imperative present tense ("add", not "added"), starts in lower case, no full stop.
- First line at most 72 characters; blank line before the body; wrap body lines at 100.
- Breaking change: `!` before the colon and/or a `BREAKING CHANGE:` footer.
- Pick the type by what the change does. `feat` and `fix` change behaviour; docs-only changes are `docs`; test-only changes are `test`.
- One logical change per commit. Never use `--no-verify`.

Examples: `feat(blocks): add a rating block`, `fix(export): wait for wired-elements before taking the screenshot`, `docs: explain how to publish a release`.

When asked for a commit message, answer with the message only.

## Code

- Node 20+, ES modules. `npm test` must pass.
- After changing anything in `src/render/blocks/`, run `npm run generate`; never edit `docs/blocks.md` or `schema/spec.schema.json` by hand.
- Do not upgrade `roughjs` (pinned to 4.3.1).

---
> Source: [mediuswareltd/openink](https://github.com/mediuswareltd/openink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
