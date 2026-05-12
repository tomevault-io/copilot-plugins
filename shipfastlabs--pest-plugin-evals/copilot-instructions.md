## pest-plugin-evals

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A PestPHP plugin (`shipfastlabs/pest-plugin-evals`) for evaluating Laravel AI SDK agents. Uses Pest's native `expect()` API — `expectAgent()` runs the agent and returns a standard Pest Expectation, so native Pest expectations work directly on agent output alongside custom LLM scorer expectations.

## Commands

```bash
composer test          # Run all checks: rector dry-run, pint lint, phpstan, pest
composer test:unit     # Pest tests only
composer test:types    # PHPStan (max level)
composer test:lint     # Pint (PSR-12)
composer test:refactor # Rector dry-run
composer lint          # Fix code style with Pint
composer refactor      # Fix with Rector
```

Run a single test file:
```bash
./vendor/bin/pest tests/Unit/Expectations/EvalExpectationsTest.php
```

Run a single test by name:
```bash
./vendor/bin/pest --filter="test name here"
```

## Architecture

### Plugin Integration (src/Plugin.php)
Implements Pest's `HandlesArguments` and `AddsOutput` contracts. Key behavior:
- `pest --eval` targets `tests/Evals/` directory (or falls back to `--group=eval`)
- Regular `pest` runs auto-add `--exclude-group=eval` so evals never mix with normal tests
- After eval runs, renders `EvalReport` summary

### Eval Execution Flow
`expectAgent()` -> `EvalExpectationContext` -> Pest `Expectation`

1. **`expectAgent()`** (src/Autoload.php): entry point. Takes agent class/closure, prompt, runs count, optional fake responses. Returns a Pest Expectation wrapping the agent output.
2. **`EvalExpectationContext`** (src/Eval/EvalExpectationContext.php): resolves the agent task (class from container, closure, or faked), executes it N times, caches outputs. Set as static `$current` so custom expectations can access prompt/agent name.
3. **Custom expectations** (src/Autoload.php): `toBeRelevant`, `toBeSafe`, `toBeFactual`, `toPassJudge`, `toBeSimilar`, `toHaveToolCalls`, `toFollowTrajectory`. Each reads context, runs the scorer, adds to EvalReport, and asserts score >= threshold.
4. **`EvalReport`** (src/Eval/EvalReport.php): singleton that collects scorer results incrementally for final output.

For single-run: `expectAgent()` returns `expect($output)` — a regular Pest Expectation.
For multi-run (`runs: N`): returns `expect([$out1,...,$outN])->each` — Pest's each proxy applies every subsequent assertion to every output.

Agents are resolved via Laravel Container (`Container::getInstance()->make()`). When `fake:` is used, the agent is bypassed entirely and predefined responses are returned.

### Scorer Interface
All scorers implement `Scorer::score(string $input, string $output, ?string $expected): ScorerResult`. Scores are 0.0-1.0.

Deterministic checks use native Pest expectations (`toContain`, `toMatch`, `toBe`, `toBeJson`, etc.) — no scorer classes needed.

**LLM-based scorers** (use `JudgesWithLlm` trait to call Laravel AI SDK): `LlmJudge`, `Factuality`, `Relevance`, `Safety`. These send a structured prompt and parse a JSON `{score, reasoning}` response.

**Other scorers**: `SemanticSimilarity` (embedding cosine similarity), `ToolCallMatch` (validates tool calls/args), `AgentTrajectory` (validates tool call sequence)

### Global Functions (src/Autoload.php)
Registers `expectAgent()` and custom Pest expectations (`toBeRelevant`, `toBeSafe`, `toBeFactual`, `toPassJudge`, `toBeSimilar`, `toHaveToolCalls`, `toFollowTrajectory`, `toPassScorer`).

### Namespace
`ShipFastLabs\PestEval\` maps to `src/`. Tests use `ShipFastLabs\PestEval\Tests\`.

## Configuration

LLM scorer provider/model configured via `config/eval.php` (published from `src/Config/eval.php`). Defaults to OpenAI `gpt-4.1-mini` for scoring and `text-embedding-3-small` for embeddings. Override with env vars: `EVAL_SCORING_PROVIDER`, `EVAL_SCORING_MODEL`, `EVAL_EMBEDDING_PROVIDER`, `EVAL_EMBEDDING_MODEL`.

## CI

GitHub Actions runs tests on PHP 8.3 + 8.4, with both `prefer-lowest` and `prefer-stable` dependency strategies. Static analysis (PHPStan + Pint) runs separately on PHP 8.3.

---
> Source: [shipfastlabs/pest-plugin-evals](https://github.com/shipfastlabs/pest-plugin-evals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
