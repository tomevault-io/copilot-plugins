## file-itr

> For AI coding agents working in this repository. Claude Code loads this file

# Agent instructions for file-itr

For AI coding agents working in this repository. Claude Code loads this file
automatically via the `@AGENTS.md` import in [CLAUDE.md](CLAUDE.md); any other
agent that reads `AGENTS.md` directly gets the same content — there is one
copy, not two.

## What this repo is

Two independent things live here — see
[README.md](README.md#repository-structure) for the full layout:

- **The skill** (`skills/itr-india/SKILL.md` + `references/`) — an agent skill
  that files Indian income tax returns. This is what actually runs when a user
  invokes the skill.
- **The engine** (`skills/itr-india/engine/`) — a separate, tested Python tax
  engine that independently verifies the skill's numbers. It is not loaded by
  `SKILL.md` and not part of the `.skill` bundle — see
  [engine/README.md](skills/itr-india/engine/README.md).

## Before you touch anything

- **Every statutory number or rule needs a verified citation.** No rule goes
  in on "this sounds right" — see
  [engine/README.md](skills/itr-india/engine/README.md) for the `Rule`/
  citation/fail-loud discipline this repo holds itself to, and apply the same
  standard to `references/*.md`.
- **Never commit real tax data** (PAN/Aadhaar, account numbers, credentials,
  Form 16/26AS/AIS/ITR JSON — real or unredacted, in a commit, test fixture,
  or PR description). See [SECURITY.md](SECURITY.md).
- **Never hand-edit or hand-commit `itr-india.skill`.** CI rebuilds it
  deterministically from `SKILL.md`/`references/`/`evals/` on every push that
  touches those paths — see `.github/workflows/skill-bundle.yml`.
- **Run the engine's tests before and after any engine change:**
  `pytest skills/itr-india/engine/tests -v`. CI runs this on every push/PR
  (`.github/workflows/engine-tests.yml`); keep it green.

## Full guidelines

[CONTRIBUTING.md](CONTRIBUTING.md) has dev setup, the commit-message
convention this repo uses, and PR expectations — read it before making a
change, not just this file.

---
> Source: [shivprime94/file-itr](https://github.com/shivprime94/file-itr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
