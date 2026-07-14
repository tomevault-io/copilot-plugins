## plex-proxy

> See [README.md](README.md)

# AGENTS.md

## Project Overview

See [README.md](README.md)

## Greenfield Project

You WILL run into situations where:
- Some new use case isn't supported by the existing APIs
- Some new feature would require significant refactors
- An existing type doesn't quite work in a new feature

In these situations, it's **ALWAYS** better to break APIs, take on big refactors, and modify types if the result would be cleaner and leaner than it would be otherwise.

**REMEMBER**: No one is using this. 

## Development

This project uses a nix development environment. All of the following commands must be prefixed with `nix develop --command`, eg: `nix develop --command ./scripts/coverage-check.sh`

- Build: `cargo build`
- Lint: `cargo clippy`
- Format: `cargo fmt`
- Rust edition: 2021
- Workspace resolver: 2

**IMPORTANT**: All new code must adhere to [`STANDARDS.md`](STANDARDS.md)

### Git Commits

**ALWAYS** do the following when asked to commit:
- Update `.gitignore` if `git status` shows files which should not be committed.
- `git add .`
- `git commit -m <commit message>`

###### Commit Messages

```
<type>[optional scope]: <description>

<body>
```

- `<type>` MUST be ONE OF: `feat`, `fix`, `chore`, `test`, `spec`, `docs`, `style`, `refactor`, `ci` or `perf`
- If provided, `scope` MUST consist of a noun describing a section of the codebase surrounded by parenthesis, eg: `fix(api)`
- `description` MUST be a short summary of code changes. It MUST NOT include task, todo, or phase references.
- `body` MUST be provided after the short description, providing additional context about the code changes. 
- `body` MUST begin one blank line after the description.
- `body` is free-form and MAY consist of any number of newline separated paragraphs.

## Documentation

- Plex Media Server (PMS) API documentation lives in `docs/api`
- `docs/api/info.md` contains general info on using and authenticating with PMS.
- Assumptions and lessons learned is kept in `docs/api/assumptions.md`.
  - Add any new assumption about PMS API behaviour to `assumptions.md` along with a recommended way to test that assumption.

## CodeGraph

In repositories indexed by CodeGraph (a `.codegraph/` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code:

- **MCP tool** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them, including dynamic-dispatch hops grep can't follow. Name a file or symbol in the query to read its current line-numbered source. If it's listed but deferred, load it by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` prints the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.

---
> Source: [inhumantsar/plex-proxy](https://github.com/inhumantsar/plex-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
