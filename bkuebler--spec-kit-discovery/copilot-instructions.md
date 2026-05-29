## spec-kit-discovery

> Context for AI assistants working on this repo. Keep this file factual; user-facing docs live in `README.md`.

# CLAUDE.md — spec-kit-discovery

Context for AI assistants working on this repo. Keep this file factual; user-facing docs live in `README.md`.

## What this is

A [spec-kit](https://github.com/github/spec-kit) extension that adds a discovery phase **before** `/specify`. Five commands (`problem` → `concept` → `clarify` → `decide` → `decompose`) produce durable artifacts on isolated `discovery/NNN-<slug>` branches in the **consumer** project. This repo *is* the extension; it is not itself a spec-kit project.

## Locked design decisions (v0.1)

Don't re-litigate these without an explicit user ask:

1. **Scope = lean + ADR.** Five commands, not three, not six. `context` and `options` are folded into `concept.md` as sections, not standalone commands.
2. **Slug strategy = AI-generated, user-confirmed, numbered branch.** Branch name `discovery/NNN-<slug>`, dir `.specify/discovery/NNN-<slug>/`. `NNN` = max(local branches, remote branches, existing dirs) + 1, zero-padded.
3. **Interaction style = draft + clarify.** Each command drafts from prose (`$ARGUMENTS`) and marks gaps with `[NEEDS CLARIFICATION: <question>]` (spec-kit's existing convention). Empty args → pure template. `clarify` resolves markers interactively.
4. **Constitution integration = opt-in promotion.** `decide` asks once: "*Does this decision establish a project-wide rule?*" Yes → append 1-3 imperative bullets to `.specify/memory/constitution.md` under `## Decisions promoted from ADRs`, each with a back-link. Never append the full ADR.
5. **Handoff = discovery branch merges to main; `/specify` runs from main per slice.** Discovery branches are conceptually a design-doc stage, not feature branches.

## Repo layout

```
extension.yml                       # manifest — 5 commands, no hooks in v0.1
commands/
  problem.md                        # branch+dir creation, 01-problem.md
  concept.md                        # 02-concept.md (options + recommendation)
  clarify.md                        # walks [NEEDS CLARIFICATION] markers
  decide.md                         # 03-adr-MMM-<slug>.md + opt-in promotion
  decompose.md                      # 04-features.md with /specify-ready prose; opt-in 05-management-summary.md, opt-in promotion of ADRs → docs/adr/ and summary → docs/management-summaries/
scripts/bash/
  discovery-context.sh              # state introspection — single source of truth
README.md                           # user-facing
CHANGELOG.md
LICENSE                             # MIT
.extensionignore                    # excluded from `specify extension add` installs
```

Each `commands/*.md` file is an AI prompt with YAML frontmatter (`description`) and a markdown body containing natural-language steps, a call to the shared script, and a template for the artifact it produces.

## Shared state script (versioned contract)

`scripts/bash/discovery-context.sh` is the **single source of truth** for repo state. All commands call it via the installed path `.specify/extensions/discovery/scripts/bash/discovery-context.sh` (spec-kit copies `scripts/` to `.specify/extensions/<ext-id>/scripts/` at install time — see `extensions/git` in spec-kit core for the reference pattern).

**Output protocol** (treat as a versioned contract — any rename is a breaking change for all 5 commands):

Header block, always emitted:

```
CURRENT_BRANCH=<name>
ON_DISCOVERY_BRANCH=yes|no
WORKING_TREE_DIRTY=yes|no
IN_GIT_REPO=yes|no
DISCOVERY_SUFFIX=<NNN-slug>          # empty if ON_DISCOVERY_BRANCH=no
DISCOVERY_DIR=.specify/discovery/<NNN-slug>   # empty if ON_DISCOVERY_BRANCH=no
NEXT_NUMBER=<NNN>                    # zero-padded, ready to use for new branches
```

Three sentinel-delimited lists follow:
```
BEGIN_EXISTING_BRANCHES_LOCAL ... END_EXISTING_BRANCHES_LOCAL
BEGIN_EXISTING_BRANCHES_REMOTE ... END_EXISTING_BRANCHES_REMOTE
BEGIN_EXISTING_DIRS ... END_EXISTING_DIRS
```

Exit code is always 0 (errors are surfaced via `IN_GIT_REPO=no`, not exit status). Bash 3.2 compatible — no `[[ ! ... =~ ... ]]`, no associative arrays. **Don't reintroduce the `[[ ! ... =~ ... ]]` pattern** — it broke once due to history-expansion escaping of `!`; `case` with glob is the established workaround.

Adding a new key is backward-compatible; renaming or removing one is not. Bump `CHANGELOG.md` accordingly when changing the protocol.

## Extension system primer (what you need to know)

- **Spec for extensions**: `extensions/EXTENSION-DEVELOPMENT-GUIDE.md` and `EXTENSION-API-REFERENCE.md` in `github/spec-kit`. Read those before changing `extension.yml` schema.
- **Manifest constraints**:
  - `extension.id` must match `^[a-z0-9-]+$` — ours is `discovery`.
  - Command names must match `^speckit\.<ext-id>\.<cmd>$`.
  - `version` is strict SemVer (`1.0.0` not `1.0` or `v1.0.0`).
- **Available hook points** (for v0.2): `before_specify`/`after_specify`, plus `_plan`, `_tasks`, `_implement`, `_clarify`, `_constitution`, etc.
- **Script path rewriting**: scripts referenced as `../../scripts/bash/foo.sh` in extension source become `.specify/scripts/bash/foo.sh` after install. We don't ship scripts yet — all helpers are inlined in command markdown.
- **Local install for testing**: `specify extension add --dev /absolute/path/to/this/repo` from inside a spec-kit project. Commands land in `.claude/commands/speckit.discovery.*.md`.

## Conventions to preserve

- **Marker format**: `[NEEDS CLARIFICATION: <specific question>]` — case-sensitive, brackets included. Don't paraphrase. `clarify.md` greps for this exact substring.
- **Filenames in discovery dirs**: `01-problem.md`, `02-concept.md`, `03-adr-MMM-<slug>.md`, `04-features.md`, optionally `05-management-summary.md`. `MMM` is per-discovery (multiple ADRs allowed); `NNN` is per-project (one per discovery).
- **Durable promotion targets** (consumer project, populated opt-in by `decompose`): `docs/adr/NNNN-<slug>.md` uses a project-wide 4-digit ADR counter independent of `MMM`; `docs/management-summaries/NNN-<discovery-slug>.md` mirrors the discovery number (one summary per discovery). Promoted ADRs have their heading rewritten to match the new `NNNN` and a back-link footer to the discovery original.
- **Bash helpers** in command bodies should be defensive (`2>/dev/null || true` where appropriate) — the consumer project's git state isn't ours to assume.
- **Idempotency**: `problem.md` reuses the current branch when invoked while already on a `discovery/*` branch. Other commands operate on whatever discovery branch is currently checked out.

## Known limitations / v0.2 backlog

In rough priority order:

1. **`before_specify` hook** + a small `speckit.discovery.check` command that warns when `/specify` runs without discovery artifacts. Omitted from v0.1 to keep the surface small.
2. **PowerShell helper parity** — currently bash-only. Spec-kit core ships `scripts/bash/` + `scripts/powershell/` siblings; add `.ps1` mirrors if/when a Windows user surfaces.
3. **Idempotency heuristic in `problem.md`** ("refinement vs new problem") is AI-judgement-based and fuzzy. May want an explicit `--new` / `--refine` flag.
4. **Constitution promotion has no dedup** — running `decide` on related ADRs can append similar rules. Document a review step or add fuzzy-match dedup.
5. **No tests yet.** The dev guide shows a pytest example using `specify_cli.extensions.ExtensionManifest`. Add a smoke test that loads `extension.yml` and asserts every `commands/*.md` referenced exists.
6. **Catalog submission** — see `extensions/EXTENSION-PUBLISHING-GUIDE.md` for the community-catalog flow when v0.1 is battle-tested.
7. **~~ADR promotion to `docs/adr/`~~** — landed in `decompose` as an opt-in prompt (along with management-summary generation and its own opt-in promotion to `docs/management-summaries/`). Open follow-up: there is no detection of re-promotion (running `decompose` twice will copy the same ADR under two different `NNNN` numbers). If this becomes a real foot-gun, add a check that grep's `docs/adr/*.md` for the discovery's source-path footer line before assigning a new number.

## Testing changes locally

1. Pick or create a spec-kit-initialized consumer project.
2. From that project: `specify extension add --dev /Users/bkuebler/Repositories/github.com/bkuebler/spec-kit-discovery`
3. Verify: `specify extension list` shows `Discovery (pre-spec workflow) v0.1.0`.
4. Invoke commands with your AI agent (e.g. `/speckit.discovery.problem <prose>` in Claude Code).
5. Inspect `.specify/discovery/NNN-<slug>/` artifacts.
6. To re-test cleanly: `specify extension remove discovery`, then re-add.

## Changelog discipline (between releases)

Keep `CHANGELOG.md`'s `[Unreleased]` section current as you commit, not retroactively at release time. The release procedure below assumes `[Unreleased]` already contains a faithful list of what's about to ship.

**For every commit that affects what a user sees or runs:** in the same commit, add a one-line entry under `[Unreleased]` in one of the Keep a Changelog categories:

- `Added` — new commands, hooks, artifacts, manifest fields, script keys.
- `Changed` — modified behavior on the existing surface (e.g. command's report text, helper script's output, README workflow guidance the user follows).
- `Fixed` — bugs in command behavior or script output.
- `Deprecated` / `Removed` / `Security` — as needed.

**Internal commits don't get an entry.** If a change touches only:

- `CLAUDE.md` (it's for AI assistants developing the extension, not users)
- Repo meta (`.gitignore`, `.extensionignore` patterns with no install impact)
- Refactors that preserve every observable behavior
- Comments / formatting

…then no `[Unreleased]` line is added. The K-a-C category filter is a useful test: if a change doesn't fit Added / Changed / Fixed / Deprecated / Removed / Security, it's probably internal.

**Release commit:** move every line from `[Unreleased]` into a fresh `[X.Y.Z] - YYYY-MM-DD` section above it; `[Unreleased]` becomes empty. Then perform the four-leg release procedure below.

## Release procedure (the four legs)

After significant changes, bump `extension.version`, add a `CHANGELOG.md` entry, create an annotated git tag `vX.Y.Z`, **and update the pinned install URL in `README.md`** — all in the same shell session. These four never drift apart:

```bash
# in the commit that bumps the version + adds the CHANGELOG entry:
git commit -m "Bump version to X.Y.Z and ..."
git tag -a vX.Y.Z -m "vX.Y.Z - <one-line summary>

<short release notes — mirror the CHANGELOG entry's headline>"
```

Use **annotated** tags (`-a -m`), not lightweight, so each release carries a message, author, and date. Tags do not push automatically — `git push --tags` (or `git push origin vX.Y.Z`) is needed.

For each new release, also append a compare-link reference at the bottom of `CHANGELOG.md` so the version header in the entry becomes a clickable diff link:

```markdown
[Unreleased]: https://github.com/bkuebler/spec-kit-discovery/compare/vX.Y.Z...HEAD
[X.Y.Z]: https://github.com/bkuebler/spec-kit-discovery/compare/vPREV...vX.Y.Z
```

Update the `[Unreleased]` link to compare from the new version each time.

The pinned install URL in `README.md` under the "Recommended: from a tagged release" section is intentional — users see exactly which version they're installing, and silent "always latest" URLs can break in surprising ways. Update it each release. Concretely, the URL pattern is:

```
https://github.com/bkuebler/spec-kit-discovery/archive/refs/tags/vX.Y.Z.zip
```

Search-and-replace `vPREV` → `vX.Y.Z` in the README before committing the release bump.

Semver bump rules for this extension:
- **Patch** (`0.1.0 → 0.1.1`): bash fixes, doc tweaks, internal refactors with stable command surface.
- **Minor** (`0.1.0 → 0.2.0`): new commands, new hooks, new artifact fields.
- **Major** (`0.x → 1.0`): renaming a command, breaking the script output protocol, changing branch naming.

## Git conventions for this repo

- Email is set locally to `b.kuebler@kuebler-it.de` (overrides global). Don't change it.
- No remote configured. Don't push or add one without an explicit ask.
- No GPG signing configured globally; do NOT add `-c commit.gpgsign=false` to commits "just in case" — pass nothing.
- Commits should be descriptive but not novel-length; the README and CHANGELOG carry the prose.

## What's NOT this repo's concern

- Spec-kit core internals (`src/specify_cli/` upstream). We only consume the extension API surface.
- The consumer project's actual feature implementation. We only produce planning artifacts.
- Hooks (until v0.2).

---
> Source: [bkuebler/spec-kit-discovery](https://github.com/bkuebler/spec-kit-discovery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
