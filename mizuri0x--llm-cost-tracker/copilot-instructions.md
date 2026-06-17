## llm-cost-tracker

> Track LLM API costs, token usage, and per-task spending across all major models.

# llm-cost-tracker

Track LLM API costs, token usage, and per-task spending across all major models.

## Installation

Install via Agent Zero Plugin Store (ZIP or Git method).

## Usage

### Log a call
```
python main.py log --model claude-sonnet-4-6 --input-tokens 1500 --output-tokens 400 --task "web scraping"
```

### Show summary
```
python main.py summary
```

### Export to CSV
```
python main.py export --output my_costs.csv
```

### List models and pricing
```
python main.py models
```

### Reset history
```
python main.py reset
```

## Agent Zero Prompt Examples

- "How much have I spent on LLM calls today?"
- "Show my cost breakdown by model"
- "Export my LLM cost history to CSV"
- "What are the top 5 most expensive tasks?"
- "Reset my LLM cost history"
- "Log 2000 input and 500 output tokens for gpt-4o for task research"

## Agent Zero Extension Hook

For automatic tracking of every LLM call, install the extension hook:
```
cp extension_hook.py /path/to/agent-zero/extensions/llm_cost_tracker_hook.py
```

Requires Agent Zero dev branch with util_model_call_after hook (PR #1215).

## Supported Models

| Model | Input ($/1M) | Output ($/1M) |
|-------|-------------|---------------|
| claude-sonnet-4-6 | $3.00 | $15.00 |
| claude-opus-4-6 | $15.00 | $75.00 |
| claude-haiku-3-5 | $0.80 | $4.00 |
| gpt-4o | $2.50 | $10.00 |
| gpt-4o-mini | $0.15 | $0.60 |
| gpt-4-turbo | $10.00 | $30.00 |
| gemini-pro-1-5 | $1.25 | $5.00 |
| gemini-flash-1-5 | $0.075 | $0.30 |
| deepseek-v3 | $0.27 | $1.10 |
| deepseek-r1 | $0.55 | $2.19 |
| llama-3-70b | $0.59 | $0.79 |
| mistral-large | $2.00 | $6.00 |

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| LLM_COST_DB_DIR | No | ~/.llm-cost-tracker/ | SQLite DB directory |
| LLM_COST_TRACKER_PATH | No | auto | Path to main.py for hook |

## Output Format

All commands output JSON for easy Agent Zero parsing.

---
> Source: [Mizuri0x/llm-cost-tracker](https://github.com/Mizuri0x/llm-cost-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
