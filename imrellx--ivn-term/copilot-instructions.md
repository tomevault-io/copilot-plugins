## ivn-term

> Personal terminal bootstrap — one script to provision a complete terminal environment on a fresh Linux box or macOS.

# ivn-term

Personal terminal bootstrap — one script to provision a complete terminal environment on a fresh Linux box or macOS.

Inspired by DHH's omaterm/omadots. Rebuilt around a four-tier package strategy: native, Homebrew, mise, and config-only.

## Build & Run

This is a shell-based project. No build step.

```bash
# Run the installer (on a fresh box)
curl -fsSL https://raw.githubusercontent.com/imrellx/ivn-term/main/install.sh | bash

# Or clone and run locally
git clone https://github.com/imrellx/ivn-term.git
cd ivn-term
./install.sh
```

## Project Conventions

### File locations

- Planning docs (PRDs, specs, QA plans): `plans/`
- Completed plans: `plans/archive/`
- Task queue (do-work): `do-work/` (auto-created by skill)
- Scratch/temp files: `tmp/` (gitignored — use for debug output, scratch scripts, test artifacts)
- Strategy and package decisions: `plans/package-strategy.md`

### Working rules

- Do NOT create scratch or temp files in the project root — use `tmp/`
- Automated commits from ralph loops use the `RALPH:` prefix
- When a plan or spec is no longer active, move it to `plans/archive/`
- Before starting implementation work, check `plans/` for an active plan related to the task. Follow the plan rather than improvising. If the plan is wrong or outdated, say so — don't silently ignore it.
- Before making architectural or design decisions, read `DECISIONS.md` to check if the question has already been decided. Do not re-litigate settled decisions without flagging it.

### Changelog (`CHANGELOG.md`)

Update `CHANGELOG.md` under the `[Unreleased]` section when making notable changes. Not every commit — only changes a human would care about when reviewing what happened. Follow [Keep a Changelog](https://keepachangelog.com) format with categories: Added, Changed, Deprecated, Removed, Fixed, Security.

### Decision log (`DECISIONS.md`)

When you make a non-trivial decision — choosing one approach over another, changing direction, adopting a pattern, dropping a dependency — log it in `DECISIONS.md`. Add a row with the date, what changed, and why. The *why* is the important part. If you can't articulate why, the decision might not be well-reasoned.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/imrellx) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
