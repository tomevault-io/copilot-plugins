## trendingai

> Auto-loaded when Claude Code starts in the cloned bundle directory. Read these once; they govern every `/trendingai` run.

# TrendingAI — project rules for Claude Code

Auto-loaded when Claude Code starts in the cloned bundle directory. Read these once; they govern every `/trendingai` run.

## First time using this bundle?

Read `README.md` cover-to-cover. The `/trendingai` skill walks you through env-var setup and the pipeline. Everything you need is there.

## Canonical entry point

`/trendingai` slash command → reads `.claude/commands/trendingai.md` → invokes the TrendingAI skill at `.claude/skills/TrendingAI/SKILL.md`.

Trigger phrases that route the same way: `run trendingai`, `let's do our AI trending report`, `AI trending report`, `fresh trendingai run`, `full harvest`.

## Hard rules for this project

1. **All synthesis happens INLINE in this Claude Code session.** Opus 4.7 (the model running this conversation) IS the synthesis layer. NO Task subagents. NO `claude --print` subprocess. NO `@anthropic-ai/sdk` imports. NO `ANTHROPIC_API_KEY` reads (the orchestrating session is the model — it authenticates via OAuth).
2. **The renderer is FROZEN.** Do not edit `src/render-features-first.ts` for cosmetic changes. Layout/visual changes are out of scope.
3. **The deterministic surfaces are FROZEN.** Data sources, harvesters, extractors (`src/features-first.ts`), clustering math, scoring formulas, `scripts/classify-one-to-watch.ts`, `scripts/fetch-captions-and-match.ts` — do not edit. They produce byte-identical output against unchanged input by design.
4. **Each run produces a NEW numbered file.** Never overwrite the existing `out/features-confirmed.html` (immutable baseline included with bundle) or any prior `out/features-confirmed-runNNN-*.html`. New runs land at `out/features-confirmed-run{NNN+1}-{anchor}.html`. Update `out/latest.html` symlink to point at the freshest.
5. **JSON-schema validation is mandatory.** After EVERY synthesis stage writes its output JSON, call `bun scripts/validate-synthesis.ts --stage <name> --file <path>`. On failure (exit 1), retry the synthesis ONCE with the same prompt/data; on second failure, mark the affected row with `error: "schema-incomplete"` and continue.
6. **Hard $ cost ceiling per run: ~$0.30.** Data-source providers + Whisper transcription. Anthropic is $0 marginal (subscription via session OAuth). Announce estimated spend at Stage 1 before harvest fires.
7. **Wall-clock target: <25 min end-to-end for fresh-harvest.** If a stage exceeds 5 min, narrate the delay; if any stage exceeds 10 min OR the pipeline exceeds 25 min total, surface the overrun and ask the user whether to continue.

## Don't do these things

- Don't run `bun run features:sentiment:legacy` or any `*:legacy` script. They exist as reference only — they use `claude --print` which lost subscription billing post-2026-06-15.
- Don't suggest installing `@anthropic-ai/sdk`, `anthropic`, or any other SDK package. The bundle has none and shouldn't.
- Don't suggest a workflow that bypasses the per-stage JSON validation. The validator catches schema drift before the renderer breaks.
- Don't overwrite `out/features-confirmed.html` (the baseline reference). Each run is archival-additive.
- Don't fabricate citations, quotes, or feature URLs. Synthesis stages cite real URLs from harvested data only; honest empty arrays beat fake content.

## Cost transparency

| Stage | Hard cost |
|---|---|
| Data harvesting + reply threads + transcription | ~$0.10-$0.20 |
| Whisper audio transcription (for video reels) | ~$0.05 |
| Anthropic Opus (synthesis) | $0 marginal (subscription) |
| **Total fresh-harvest run** | **~$0.15-$0.30** |
| Render-only path | $0 |

## What "done" looks like per run

1. Pipeline narrated stage-by-stage with timestamp + count + status.
2. JSON validated after every synthesis stage.
3. `out/features-confirmed-run{NNN}-{anchor}.html` written with run-incremented N.
4. `out/latest.html` symlink updated.
5. Report opens in user's browser via `interceptor open` (or the absolute path is printed for manual open if Interceptor is unavailable).
6. Final narration summarizes: run number, output path, size, content counts, wall-clock, hard $ spend.

---
> Source: [Glitch-Cat-Club/TrendingAI](https://github.com/Glitch-Cat-Club/TrendingAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
