## dev-compact-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **design-and-build repo for a reusable Claude Code "dev workspace" standard**. The deliverable is a committable bundle of `.claude/skills/dev-*` skills, `.claude/hooks/*.py` Python hooks, and a `.claude/settings.json` that other projects drop in to get:

1. A `.dev/` local workspace convention (gitignored, never source-of-truth).
2. A user-driven checkpoint/compact flow that **replaces** auto-compact.

## Source of truth

The bundle itself is the spec. Read these in order before making changes:

1. `.claude/settings.json` — DISABLE_AUTO_COMPACT, statusLine, the four hook events.
2. `.claude/hooks/*.py` — the actual behavior. In particular, `ensure-dev-workspace.py:DEV_DIRS` and `:GITIGNORE_LINES` are the canonical `.dev/` layout and gitignore set.
3. `.claude/skills/*/SKILL.md` — user-facing rules.

The original design notes that motivated this bundle live in `.dev/docs/spec/req-1.md` and `req-2.md` (local-only, not committed). They are historical context, not authoritative — when the spec and the code disagree, the code wins.

## Core design invariants

These are non-negotiable — they motivate the entire architecture:

- **No auto-compact.** Settings must include `"env": { "DISABLE_AUTO_COMPACT": "1" }` and a `PreCompact` hook with `matcher: "auto"` that returns `decision: "block"` as a belt-and-suspenders guard. Manual `/compact` must remain available.
- **Two-tier offer, don't act.** Two thresholds on `used_percentage` (defaults: `CLAUDE_HANDOFF_SOFT_THRESHOLD=25`, `CLAUDE_HANDOFF_HARD_THRESHOLD=30`). At ≥25% the status line writes the flag with `offer_level="soft"` and the `Stop` hook emits `decision:"block"` with three options (`/dev-checkpoint`, `/compact`, `continue without compact`). At ≥30% the level escalates to `"hard"` and the hook emits a stronger English message that omits the `continue without compact` option. Never compact mid-tool-loop. The `used_percentage` is read from the payload's `context_window.used_percentage`, falling back to `100 - remaining_percentage` if absent.
- **Hysteresis on the offer.** `REMIND_STEP=5` buckets — once offered at bucket N, don't re-offer until bucket N+5. Bucket boundaries (25, 30, 35, …) align with the two thresholds so a soft offer at bucket 25 naturally escalates to hard at bucket 30. State fields: `last_offered_bucket`, `current_bucket`, `offer_bucket`, `offer_level`, `needs_compact_offer`, `soft_threshold`, `hard_threshold`.
- **`.dev/` is never committed.** The `SessionStart` hook must idempotently append `.dev/`, `.development/`, `playwright-report/`, `test-results/`, `blob-report/` to `.gitignore` if missing.
- **Repository state is authoritative.** Handoff summaries and continuation prompts in `.dev/handoffs/` are evidence/context, not source of truth. The continuation prompt template explicitly says so.
- **Skills with side effects use `disable-model-invocation: true`** (e.g., `dev-init`, `dev-checkpoint`, `dev-clean`) so Claude doesn't fire them autonomously. `dev-workspace` is the only ambient skill.

## Target structure to produce

```
.claude/
├── settings.json                         # DISABLE_AUTO_COMPACT, statusLine, all 4 hook events
├── hooks/
│   ├── ensure-dev-workspace.py           # SessionStart: mkdir tree, seed README, patch .gitignore
│   ├── context-statusline.py             # statusLine: writes .dev/state/context-state.json
│   ├── offer-manual-compact.py           # Stop: emits decision:"block" once per bucket; soft text at >=25%, hard text at >=30%
│   ├── block-auto-compact.py             # PreCompact matcher:"auto": always block
│   ├── save-compact-handoff.py           # PostCompact matcher:"manual": writes summary+prompt
│   └── inject-latest-handoff.py          # SessionStart matcher:"compact|clear|resume": stdout injection
└── skills/
    ├── dev-workspace/SKILL.md            # ambient convention doc
    ├── dev-init/SKILL.md                 # disable-model-invocation
    ├── dev-checkpoint/SKILL.md           # disable-model-invocation
    ├── dev-research/SKILL.md
    ├── dev-review/SKILL.md
    ├── dev-artifact/SKILL.md
    └── dev-clean/SKILL.md                # disable-model-invocation
```

The `.dev/` tree the `ensure-dev-workspace.py` hook creates lives in that hook's `DEV_DIRS` constant — that's the single source of truth; don't duplicate the list elsewhere.

## Hook I/O contract (easy to get wrong)

- All hooks read JSON from **stdin**. Resolve project root as `os.environ["CLAUDE_PROJECT_DIR"] or payload["cwd"] or os.getcwd()`.
- The status line command is invoked as a shell command and must print one short line to stdout — its side effect is writing `.dev/state/context-state.json`. Read `payload["context_window"]["used_percentage"]`, falling back to `100 - payload["context_window"]["remaining_percentage"]`; if neither is present, print `context: n/a` and exit 0.
- `Stop` hook must check `payload.get("stop_hook_active")` and bail early — otherwise it loops.
- Hook responses use `print(json.dumps({...}))`. To halt Claude: `{"decision": "block", "reason": "..."}`. To inject context (SessionStart): print plain text to stdout, no JSON envelope.
- JSON writes go through a `tmp` file then `os.replace()` for atomicity — see `write_json` in `offer-manual-compact.py` (same pattern reused across hooks).

## Slash commands vs skills

The checkpoint flow ships only as a skill (`.claude/skills/dev-checkpoint/SKILL.md` → `/dev-checkpoint`). Do not add a parallel slash command under `.claude/commands/` — they would collide.

## Out of scope

- Don't invent build/test tooling beyond pytest. The Python hooks are stdlib-only; tests use pytest (`pip install pytest`).
- Don't pre-create `.dev/` subdirectories in this repo — they're created at runtime in *consumer* projects by the `ensure-dev-workspace.py` hook.
- User-facing prompts in `offer-manual-compact.py` are in English (two templates: `SOFT_BODY`, `HARD_BODY`) and shown verbatim.

---
> Source: [miolamio/dev-compact-skills](https://github.com/miolamio/dev-compact-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
