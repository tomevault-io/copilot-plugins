## pagekit

> Neutral agent contract for PageKit. Readable by Codex, Claude Code, and any agent reading the repo cold.

# AGENTS.md

Neutral agent contract for PageKit. Readable by Codex, Claude Code, and any agent reading the repo cold.

## What PageKit is

A Claude Code plugin plus a chat-paste path for producing web pages that are contextually relevant to your audience and what your product does. The method runs upstream of copy: signal → spine → first-page decision → argument shape → proof map → first-page draft → claim check.

## If you are Claude Code

The plugin is self-contained in this repo. Skills live under [`skills/`](skills/), subagents under [`agents/`](agents/), and the plugin manifests under [`.claude-plugin/`](.claude-plugin/). Invoke the orchestrator with `Run PageKit on <object>` and it dispatches the 7-step chain.

Key entry points:
- `skills/pagekit/SKILL.md`: orchestrator
- `skills/pagekit-<step>/SKILL.md`: per-step skills (signal-doc through claim-check)
- `agents/pagekit-claim-checker.md`: adversarial claim-check subagent
- `agents/pagekit-evaluator-pass.md`: adversarial evaluator-pass subagent

## If you are Codex or another chat-based agent

Use the prompt-paste path. Open [`prompt-paste/`](prompt-paste/) and walk the 7 prompts in order. Each step consumes the previous step's output.

## Running a fully-logged run from the shell

```sh
# scaffold the folder
bash scripts/new-run.sh <run-name>

# …work through prompts/NN-*.md and outputs/NN-*.md

# validate against the logging tiers
bash scripts/run-check.sh runs/<run>

# claim-check the draft at the severity you intend
bash scripts/claim-check.sh runs/<run>/first-page-draft.md runs/<run>/proof-map.md --severity hard

# heuristic anti-slop regression against the draft
bash scripts/slop-check.sh runs/<run>/first-page-draft.md

# pre-flight check for the repo
bash scripts/doctor.sh
```

## Hard rules

- **Do not hard-code homepage-first.** Step 03 picks the first page from the object. Name rejected candidates.
- **Do not invent proof.** Mark unverified product-specific claims with `*[verification flag: ...]*` inline.
- **Do not ship a draft with AI-slop patterns.** See `skills/pagekit/references/anti-slop.md`. Run `scripts/slop-check.sh` and resolve every hit.
- **Do not skip run logging.** Fully-logged runs are defined in `skills/pagekit/references/run-logging.md`. `scripts/run-check.sh` tells you if your run meets the tier.

## Getting unstuck

- Output of a step feels generic → fix upstream source material, not the step output.
- First-page decision reluctant → add a mechanism / proof / comparison brief in `sources/` and rerun.
- Claim-check flagging the same pattern every run → update `skills/pagekit/references/anti-slop.md` and the argument-shape drafting constraints so the pattern does not return.

---
> Source: [hnshah/pagekit](https://github.com/hnshah/pagekit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
