## fterm

> - Think in English, explain, and respond to chat in Japanese.

# Project Summary

- Think in English, explain, and respond to chat in Japanese.
- Use half-width brackets instead of full-width brackets in the Japanese explanations output.
- When writing Japanese and half-width alphanumeric characters or codes in one sentence, please enclose the half-width alphanumeric characters in backquotes and leave half-width spaces before and after them.

## Commands

All tasks use `mise run <task>`:

| Task                  | Command                                                |
| --------------------- | ------------------------------------------------------ |
| Setup                 | `mise run setup`                                       |
| Build                 | `mise run build`                                       |
| Build (release)       | `mise run build:release`                               |
| Build (timings)       | `mise run build:timings`                               |
| Check                 | `mise run check`                                       |
| Test                  | `mise run test`                                        |
| TDD watch             | `mise run test:watch`                                  |
| Doc tests             | `mise run test:doc`                                    |
| Trace test            | `mise run test:trace`                                  |
| Format                | `mise run fmt`                                         |
| Format check          | `mise run fmt:check`                                   |
| Lint (clippy)         | `mise run clippy`                                      |
| Lint strict           | `mise run clippy:strict`                               |
| Lint                  | `mise run lint`                                        |
| Lint (GitHub Actions) | `mise run lint:gh`                                     |
| AST rules             | `mise run ast-grep`                                    |
| Pre-commit (required) | `mise run pre-commit`                                  |
| Pre-push              | `mise run pre-push`                                    |
| Coverage              | `mise run coverage`                                    |
| Coverage (HTML)       | `mise run coverage:html`                               |
| Audit                 | `mise run audit`                                       |
| Deny (licenses/deps)  | `mise run deny`                                        |
| Miri (UB detection)   | `mise run miri`                                        |
| Build (OTel)          | `cargo build --features otel`                          |
| Clean (full)          | `mise run clean`                                       |
| Clean (sweep)         | `mise run clean:sweep`                                 |
| Clean (cache)         | `mise run clean:cache`                                 |
| Badges (init)         | `mise run badges:init`                                 |
| Claude Code (install) | `mise run claudecode:install`                          |
| O2 (install)          | `mise run o2:install`                                  |
| O2 (start)            | `mise run o2`                                          |
| O2 (stop)             | `mise run o2:stop`                                     |
| CodeQL (tool install) | `mise install --config-file .mise/tools.optional.toml` |
| CodeQL (db + packs)   | `mise run codeql:install`                              |
| CodeQL (analyze)      | `mise run codeql`                                      |
| SSH test setup        | `mise run ssh:setup`                                   |
| sshd (test)           | `mise run sshd`                                        |
| Traefik setup (WSL2)  | `mise run traefik:setup`                               |
| Dev container up      | `mise run dev:up`                                      |
| Dev container down    | `mise run dev:down`                                    |
| Dev container exec    | `mise run dev:exec`                                    |
| Dev container status  | `mise run dev:status`                                  |

## Commit Convention

Conventional Commits: `<type>: <description>` or `<type>(<scope>): <description>`

Allowed types: feat, update, fix, style, refactor, docs, perf, test, build, ci, chore, remove, revert

## Workflow

1. Write tests (for new features / bug fixes)
2. Implement
3. Run `mise run test` — all tests must pass
4. Stage only the relevant files
5. Run `mise run pre-commit` (runs clean:sweep, fmt:check, clippy:strict, ast-grep, lint:gh)
6. If errors, fix → re-stage → re-run `mise run pre-commit`

## Code Comments

- Write all code comments (doc comments, inline comments) in concise English.

## Key Coding Rules

- **Imports**: All `use` statements at file top level, grouped: `std` -> external crates -> `crate`/`super`. No wildcards (`*`). Aliases (`as`) permitted for name conflicts and re-exports.
- **Error handling**: Never use bare `?`. Always add `.context()` or `.with_context()`.
- **Logging**: Use `tracing` crate, not `println!` / `dbg!`. For container/OTel support, build with `--features otel` and set `OTEL_EXPORTER_OTLP_ENDPOINT` env var.
- **Tests**: Arrange / Act / Assert pattern. Unit tests in `#[cfg(test)] mod tests`, integration tests in `crates/<name>/tests/`. `#![allow(clippy::unwrap_used)]` is permitted in test code.
- See [docs/project_rules.md](./docs/project_rules.md) for full details.

## Skill Maintenance

- When modifying coding rules, workflow, or project conventions in `CLAUDE.md` or `docs/project_rules.md`, also update the corresponding `.claude/skills/` files to keep them in sync.

---
> Source: [naa0yama/fterm](https://github.com/naa0yama/fterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
