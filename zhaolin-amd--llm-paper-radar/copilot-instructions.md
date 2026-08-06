## llm-paper-radar

> Repo-level guidance for coding agents (Claude Code, etc.) operating in this repo.

# AGENTS.md — llm-paper-radar

Repo-level guidance for coding agents (Claude Code, etc.) operating in this repo.

## Don't silently skip expensive pipeline steps

When running `scripts/daily.sh`, `scripts/snapshot.sh`, or any wrapper, **do
not** add `PAPER_RIVER_SKIP=1` (or `PAPER_RIVER_MAX=<small N>`, or any
other knob that disables/throttles work the user didn't ask to skip) on
your own initiative — even when you think it'll take too long.

The `auto_paper_river` step runs the `ljg-paper-river` skill in headless
mode for every paper surfaced in the current rollup window (default
`--window-days 1`, matching `daily.sh`'s `--days`). Typical cron runs
generate a handful of `.org` files — but a `--all-history` backfill or
a long `--days` window can balloon to hundreds × 5-10 min each. The
output `.org` files are the whole point of the run.

If you're worried about runtime, **ask** before skipping:

> "auto_paper_river will run /ljg-paper-river per paper for N papers
> (~M minutes each, ~T total). Run it / cap at K / skip entirely?"

Use `AskUserQuestion` with options like:
- Run all (default — matches cron behavior)
- Cap at N this run (set `PAPER_RIVER_MAX=N`)
- Skip this step (set `PAPER_RIVER_SKIP=1`)

Same principle applies to anything else in this repo: don't disable
fetch, dedupe, summarize, translate, render, push, or cron-equivalent
behavior unless the user explicitly asks. This includes `BACKFILL_EMPTY_SKIP=1`
(skips the empty-`arxiv.json` recovery sweep in `daily.sh`) — leave it on
unless the user opts out. The cron-driven invocations
(`cron/daily_wrapper.sh` etc.) intentionally run the full pipeline with
no caps; manual runs should default to the same unless the user opts out.

## Manual reruns

When the user says "rerun", "force", or "redo today" — match what cron
would do. Pass `--force` if needed for idempotency, but keep all the
heavy steps enabled by default.

## Pull before you push

**Always `git pull` (fast-forward, or `--rebase` if you have local commits) before
`git push`** — never push straight from a local checkout that hasn't just synced with
`origin/main`. `daily.sh`/`weekly.sh`/`rollup.sh` already do this internally
(`git pull --rebase --autostash` before their own commit+push); apply the same discipline
to any git action you run manually outside those scripts.

- **Why:** caught in practice on a sibling repo in the same working tree — a session
  committed and nearly pushed on a base 24 commits behind `origin/main`. This repo's cron
  scripts push frequently (daily digest, weekly rollup), so a manual checkout goes stale
  fast.
- **How to apply:** before any manual `git push`, run `git fetch` + check
  `git log HEAD..origin/main`; if behind, pull first. Applies mid-session too — cron may
  have pushed a new digest while you were working.

---
> Source: [zhaolin-amd/llm-paper-radar](https://github.com/zhaolin-amd/llm-paper-radar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
