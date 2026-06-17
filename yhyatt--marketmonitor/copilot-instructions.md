## marketmonitor

> Weekly AI market intelligence service for Yonatan Hyatt (AI Strategist, Siemens).

# Market Monitor

Weekly AI market intelligence service for Yonatan Hyatt (AI Strategist, Siemens).

## What it does

1. Scans arXiv (cs.AI, cs.LG, cs.CL, cs.MA), HuggingFace, GitHub, and AlphaSignal
2. Filters with two-stage pipeline: keyword pre-filter + LLM relevance scoring
3. Logs to `memory/market/` JSONL files
4. Delivers weekly digest via Telegram + email

## Usage

```bash
# Scan all sources for new items
python3 -m market_monitor scan

# Generate and send digest
python3 -m market_monitor digest --telegram 5553808416 --email hyatt.yonatan@gmail.com

# Full pipeline (scan + digest)
python3 -m market_monitor run --telegram 5553808416 --email hyatt.yonatan@gmail.com

# Check status
python3 -m market_monitor status

# Dry-run scan
python3 -m market_monitor test
```

## Cron

Weekly digest, Sunday 6 PM:

```
python3 -m market_monitor run --telegram 5553808416 --email hyatt.yonatan@gmail.com
```

## Environment

Required:
- `ANTHROPIC_API_TOKEN` - For LLM scoring

Optional:
- `GITHUB_TOKEN` - For higher GitHub API rate limits
- `GOG_KEYRING_PASSWORD` - For email sending and AlphaSignal Gmail access

## Themes tracked

- Agentic AI, multi-agent systems, orchestration
- Foundation model positioning, scaling laws
- Paradigm shifts, emergent capabilities
- Enterprise AI, AI governance, alignment
- Harness engineering, context management
- Reasoning models, chain-of-thought, tool use

## Data stores

- `memory/market/papers.jsonl` - Scored papers
- `memory/market/hf_releases.jsonl` - HuggingFace items
- `memory/market/github_signals.jsonl` - GitHub velocity signals
- `memory/market/github_baseline.json` - Star count baseline

## Digest format

Telegram (compact):
```
🧠 AI Market Pulse — Apr 7

📄 [Title] — [thesis]
   arxiv.org/... ⚡ [signal]

📦 HF: [Model] — [description]
🚀 GitHub: vllm-project/vllm +1240★ (+8.2%)

[Why this week matters paragraph]
```

Email: Rich HTML with sections for Papers, HuggingFace, GitHub, and Synthesis.

---
> Source: [yhyatt/MarketMonitor](https://github.com/yhyatt/MarketMonitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
