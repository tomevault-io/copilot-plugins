## pywatershed

> <!-- START doctoc generated TOC please keep comment here to allow auto update -->

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [pywatershed guidance for Claude](#pywatershed-guidance-for-claude)
  - [Testing](#testing)
  - [Conventions](#conventions)
  - [Branches and releases](#branches-and-releases)
  - [Skills](#skills)
  - [Personal instructions](#personal-instructions)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# pywatershed guidance for Claude

pywatershed is a Python package for hydrologic modeling, a
reimplementation of PRMS process representations (see README.md).

## Testing

- Tests live in `autotest/` and are run from that directory. Tests using
  the `simulation` fixture require `--domain` (e.g.
  `pytest test_prms_atmosphere.py --domain drb_2yr`) and optionally
  `--control_pattern`; there is no default domain.
- Domain test data must be generated before testing; see DEVELOPER.md.
- `autotest/ci_local.sh` runs the CI suites locally and must be kept in
  sync with `.github/workflows/ci.yaml` (which test files each job runs
  or ignores).
- CI uses `--error-for-skips`: a test that skips conditionally must be
  either `--ignore`d in the broad CI steps or run in a dedicated step
  whose `--control_pattern` avoids the skip.

## Conventions

- Lint and format with `ruff check .` and `ruff format .` (pre-commit
  also runs doctoc for markdown TOCs).
- User-facing changes get an entry in `doc/whats-new.rst`; use
  `(:pull:`XXX`)` as the placeholder until the PR number exists.
- New public classes are exported in `pywatershed/__init__.py` (and the
  subpackage `__init__.py`) and listed in `doc/api/*.rst`.

## Branches and releases

- `main` always holds the latest release; `develop` holds the next one
  and is the base for feature branches.
- Flows between `main` and `develop` use merge commits, never squash.
- Releases follow `.github/RELEASE.md`; the `/release` skill assists
  step by step. `.github/scripts/release_preflight.sh` checks release
  version consistency locally and in CI.

## Skills

Project-specific skills live in `.claude/skills/` and are invoked as
slash commands in a Claude session. Keep this list current when adding
skills:

- `/release` — assists a pywatershed release step by step, following
  `.github/RELEASE.md`: drafts the commands for the human to run,
  runs the checks, and verifies each stage (PyPI, tag, release assets).

Claude: in your first reply of a session, briefly show the user this
bullet list of available skills (many users don't know skills exist).
Keep it short and then address their request; don't let the list crowd
out their actual question.

## Personal instructions

Individual contributors may keep personal instructions (untracked) at
the path below; if the file does not exist the import is skipped.

@~/.claude/pywatershed-instructions.md

---
> Source: [DOI-USGS/pywatershed](https://github.com/DOI-USGS/pywatershed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
