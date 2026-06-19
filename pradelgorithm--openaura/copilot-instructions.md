## openaura

> This file gives coding agents the project-specific context they need to work safely in

# AGENTS.md

This file gives coding agents the project-specific context they need to work safely in
Open AURA. It complements the human-facing README and contribution docs.

## Project Overview

Open AURA is a Python package and CLI for generating evidence-based weekly project
briefs from delivery signals. The core implementation lives in `openaura/`, tests live
in `tests/`, and bundled instructions/templates live under `openaura/instructions/` and
`openaura/templates/`.

## Setup Commands

- Create a virtual environment: `python -m venv .venv && source .venv/bin/activate`
- Install for development: `python -m pip install -e ".[dev]"`
- Run the CLI locally: `aura --help`

## Required Checks

Run the smallest relevant check while iterating, then run the full set before opening a
PR that changes code:

- Lint: `python -m ruff check .`
- Format check: `python -m ruff format --check .`
- Type check: `python -m mypy openaura`
- Security lint: `python -m bandit -r openaura -ll`
- Tests and coverage: `python -m pytest --cov=openaura --cov-fail-under=80`
- Dependency audit: `python -m pip_audit --skip-editable`

For docs-only changes, at minimum review the rendered Markdown where practical and run
`git diff --check`.

## Code Style

- Target Python 3.11 and newer.
- Use type hints and keep `mypy` strict-compatible.
- Use `pathlib` for filesystem paths.
- Keep connector failures graceful: return warnings instead of crashing the pipeline.
- Do not add raw provider SDK calls for LLMs; all model interactions go through
  Pydantic AI.
- Keep dependencies small and justified. Prefer the standard library or existing
  project dependencies.

## Security Rules

- Never commit secrets. Config files may name environment variables but must not contain
  token values.
- Do not log secrets, credentials, personal data, or raw third-party API payloads unless
  they have been deliberately redacted.
- Use `https://` for connector base URLs.
- Prefer safe Python APIs. Do not use `exec` or `eval` on untrusted input, and avoid
  shell execution unless there is a clear need and arguments are safely structured.
- When adding dependencies, verify that the package exists, is maintained, and is
  compatible with the Apache-2.0 license and this project's supply-chain expectations.

## Agentic Development Policy

Open AURA allows AI-assisted and agentic development, but a human contributor remains
responsible for every submitted change.

- Human review is required before merge for all agentic changes.
- AI agents must not add DCO `Signed-off-by` trailers on behalf of a person. Only the
  human submitter may certify the DCO.
- If a substantial change was AI-assisted, disclose it in the PR body or an
  `Assisted-by:` commit trailer, for example:
  `Assisted-by: Codex:gpt-5.4`.
- The human submitter must understand the change well enough to explain what it does,
  why it is needed, how it was tested, and how it affects security and licensing.
- Human reviewers must explicitly consider license compatibility and security impact for
  agentic changes, especially new dependencies, generated code, CI workflows, release
  automation, and connector/authentication logic.
- Do not submit generated code that you cannot review, test, maintain, and defend.
- Treat AI suggestions like contributions from an unfamiliar collaborator: useful, but
  not authoritative.

## License and Security Review

The repository is Apache-2.0 licensed. Do not alter `LICENSE` unless the maintainers
explicitly request a legal/license update.

For AI-assisted changes:

- Confirm generated content does not include third-party code with incompatible license
  terms.
- Preserve required copyright, attribution, SPDX, and notice information when using
  third-party material.
- Prefer original implementations over copied snippets.
- Include a security review note in the PR when changing authentication, secret handling,
  CI/CD, release, networking, file I/O, or dependency behavior.

## PR Expectations

- Keep PRs focused and small enough for a human to review.
- Include a clear summary, testing notes, and any AI-assistance disclosure.
- Update tests when behavior changes.
- Update docs when user-facing behavior, CLI flags, configuration, or security posture
  changes.
- Do not leave unrelated formatting churn in the diff.

## Reference Guidance

These instructions are informed by:

- AGENTS.md: https://agents.md/
- Linux kernel AI coding assistant guidance:
  https://docs.kernel.org/process/coding-assistants.html
- Linux Foundation generative AI policy:
  https://www.linuxfoundation.org/legal/generative-ai
- OpenSSF security-focused guide for AI code assistant instructions:
  https://best.openssf.org/Security-Focused-Guide-for-AI-Code-Assistant-Instructions
- Red Hat guidance on AI-assisted open source contribution:
  https://www.redhat.com/en/blog/accelerating-open-source-development-ai

---
> Source: [pradelgorithm/openaura](https://github.com/pradelgorithm/openaura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
