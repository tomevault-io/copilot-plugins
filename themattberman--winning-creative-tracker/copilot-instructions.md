## winning-creative-tracker

> This public kit is self-contained. Operate it as a market-evidence diagnostic kit, not as a campaign-production system. The report can point qualified users to the paid StealAds/Emerald creative lane.

# winning creative tracker agent guide

This public kit is self-contained. Operate it as a market-evidence diagnostic kit, not as a campaign-production system. The report can point qualified users to the paid StealAds/Emerald creative lane.

---

## front door

The `diagnostic-report` skill is the front door for live operation. It should:

1. Take a market niche and optional competitor seeds.
2. Pull real organic and paid creative with the user's BYOK services.
3. Save a validated capture under a run directory as `raw/*.json`.
4. Record source counts, cost rows, and skipped coverage in `raw/run_meta.json`.
5. Run the deterministic engine:

```bash
python -m cie diagnostic-report --config <config.toml> --fixtures <run-dir> --out <run-dir>
```

The engine lives in `cie/`. The agent-facing skill lives in `skills/diagnostic-report/`.

---

## hard rules

- Stop at the diagnostic HTML report.
- Keep the report CTA pointed at the paid-work intake path. Do not replace it with a generic local-agent CTA.
- The source swipe file may show pulled posts and ads from `raw/*.json`; it must not generate new hooks, scripts, concepts, or ad copy.
- Do not write hooks, scripts, shot lists, media plans, campaign recommendations, or content assets unless a separate private/internal kit explicitly owns that job.
- Start with the offline example when validating a fresh install.
- Use only real sampled creative in the capture. Never fabricate posts, ads, metrics, actor reads, competitors, screenshots, or URLs.
- Never invent a gap. If the CLI reports `gap: none (...)`, write `No gap found yet` and recommend the next useful pull: more competitors, wider organic sample, or an adjacent niche.
- Log estimated pull cost and actual API calls when running live.
- Honor the configured `llm.tier`: `subscription` uses the user's local agent subscription path; `openrouter` and `api_key` require a model and key env var.
- Never invent environment variables. Live pulls use `SCRAPECREATORS_API_KEY` and `VIRLO_API_KEY`. Optional headless LLM use can read `OPENROUTER_API_KEY` or another explicitly configured `llm.api_key_env`.
- Keep outputs screenshot-able and evidence-backed.

---

## dev and public split

Public-kit work happens inside this repository only. Do not depend on external private repositories, private plugins, private skills, local absolute paths, or client-specific context.

The public contract is:

- `cie/` contains the deterministic report engine.
- `skills/diagnostic-report/` contains the bundled agent operator.
- `tests/fixtures/diagnostic-run-example/` contains the zero-key example capture.
- `examples/` may contain rendered public examples.
- BYOK services are external and user-provided.

---

## validation checklist

Before calling a run complete:

1. Run the offline example render.
2. Confirm `report.html` exists.
3. Confirm every claim in the report traces to `raw/*.json`.
4. Confirm no private paths or private context are present.
5. Confirm live runs name API cost, skipped coverage, and missing env vars plainly.

If a check fails, fix the artifact or report the blocker. Do not smooth it over in prose.

---
> Source: [TheMattBerman/winning-creative-tracker](https://github.com/TheMattBerman/winning-creative-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
