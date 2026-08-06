## ssot-skill

> > AGENTS.md should fit in the reader's working memory; if a section grows past comfortable reading, consolidate or demote before adding.

# AGENTS.md

> AGENTS.md should fit in the reader's working memory; if a section grows past comfortable reading, consolidate or demote before adding.

## What This Repo Is

SSOT-SKILL is a lifecycle skill bundle for software repositories. It organizes long-lived repository facts into a Markdown `SSOT/` system. The bundle contains six skills:

| Skill | Role |
|-------|------|
| `ssot-preflight` | Read repository SSOT before substantive work |
| `ssot-bootstrap` | Create or continue repository SSOT structure |
| `ssot-closeout` | Update SSOT after a substantive change batch |
| `ssot-audit` | Catch up historical commits, sessions, or protocol versions |
| `ssot-doctor` | Health check, stop review, CORE-REF, ADAPTER, CONSUMPTION |
| `ssot-skill` | Compatibility shim; routes to one of the five above |

## Install

When a user asks you to install or reinstall this bundle, follow [`INSTALL.md`](./INSTALL.md). The short form:

```bash
curl -fsSL https://raw.githubusercontent.com/huangpufan/SSOT-SKILL/main/install.sh | bash -s -- --quickstart
```

For upgrade or uninstall, see `bash install.sh --help`.

## Usage

After install, restart the agent session so skills are discovered. Then use:

- `$ssot-preflight` — before starting any substantive repository work
- `$ssot-bootstrap` — when `SSOT/` is missing or bootstrap is incomplete
- `$ssot-closeout` — before final response, `claim_done`, or commit
- `$ssot-audit` — when the user asks to sync commits/sessions/protocol
- `$ssot-doctor` — for health checks or stop review before high-impact claims

## Language

- Skill protocol text (`SKILL.md`, `references/`) is in English for routing accuracy.
- Bootstrap templates ship in both `en/` and `zh/`; the installer copies one language based on `--lang`.
- Consumer-facing `SSOT/` documentation follows the consumer repo's `documentation_language` lock.

## Repo Boundaries

- `install.sh` is the installer entrypoint; do not modify unless the skill list or agent registry changes.
- `skills/*/SKILL.md` is the protocol authority for each skill.
- `skills/ssot-bootstrap/assets/templates/{en,zh}/` contains template files; keep `en/` and `zh/` in file-name parity.
- `skills/*/agents/openai.yaml` must stay in sync with each skill.
- `VERSION` and `skills/ssot-preflight/SKILL.md` `metadata.protocol_version` must match (CI enforces).
- `CHANGELOG.md` follows Keep a Changelog format.

## Commit Discipline (Global)

This section applies to **maintainers operating against `origin/main` directly**. External contributors follow [`CONTRIBUTING.md`](./CONTRIBUTING.md) — that is the single entry point for community PRs.

Every modification to this repository — code, templates, docs, installer, tests — MUST end with `git commit && git push` to `origin/main` as the final step of the task. Do not leave changes in the working tree at the end of a turn.

- Direct commits to `main` are the established pattern for maintainers on this repo; external contributors go through `CONTRIBUTING.md` (branch + PR).
- Group related edits into a single commit per task; do not split into per-file commits.
- Commit message follows the `<type>: <subject>` form used in `git log` (e.g. `docs:`, `feat(protocol):`, `fix:`, `vX.YY (low|medium|high):` for protocol bumps).
- Run any relevant local check before committing (`bash tests/test-bundle-shape.sh` for shape changes; `bash skills/ssot-doctor/assets/scripts/test/run-tests.sh` for lint changes); a failing check is a blocker, not a footnote.
- If the user explicitly says "don't push" or "stage only", that overrides this rule for that turn only — it does not change the default.

## Writing Style (Global)

KISS is the permanent SSOT design principle: choose the shortest **reliable path to understanding**, not the fewest words. Keep one fact in one owner and make tables serve prose rather than replace it; never compress away the context, causal story, boundary, current path, state/trust/failure behavior, or current-versus-target distinction a cold reader needs. Every section of every user-facing SSOT body file is first written for the stranger who lands tonight knowing nothing. The default reader is an implementation delegator who may not write or read code: lead with the scene, decision, delegated action, visible result and fitting evidence, then the stop or escalation boundary; explain unavoidable terms before exact tokens, commands, paths, or codes. Tables, codes and tags the reader cannot interpret are not paragraphs. The scope is every SSOT area the bundle generates — `product/`, `architecture/` (root, views, domains), the process root plus `development/`, `testing/`, `benchmark/`, deployment, `release/`, applicable operations/security owners, the records root plus `decisions/`, research, `gotchas/`, `bugs/`, `tech-debt/`, `glossary/`, and root `SSOT/README.md` — plus register-only discipline for `STATUS.md` / `.bootstrap/`: cells stay pointer-sized and narrative moves to its owner. `ssot-preflight/references/reader-quality.md` is the unique owner of the shared writing floor, all reader-area completeness profiles, and the product/architecture structured comprehension contract; `ssot-bootstrap` §3.7 is only a routing adapter.

## Key Files

| File | Purpose |
|------|---------|
| `install.sh` | Bundle installer (supports 70+ agents) |
| `VERSION` | Bundle protocol version |
| `CHANGELOG.md` | Version history |
| `skills/*/SKILL.md` | Skill protocol and instructions |
| `skills/*/agents/openai.yaml` | Agent metadata for each skill |
| `skills/*/references/` | Reference material loaded by skills at runtime |
| `skills/ssot-bootstrap/assets/templates/{en,zh}/` | Bootstrap templates |
| `skills/ssot-doctor/assets/scripts/ssot-lint.sh` | Deterministic SSOT lint |
| `tests/` | Bundle shape and installer e2e tests |
| `.github/workflows/ci.yml` | CI pipeline |

---
> Source: [huangpufan/SSOT-SKILL](https://github.com/huangpufan/SSOT-SKILL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
