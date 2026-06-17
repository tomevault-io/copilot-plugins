## brave-for-ai-edge-gallery

> Search the web using Brave Search API to find current information, news, and real-time data beyond the model's training knowledge. Use this skill whenever the user asks to search the web, look up recent news, find current information, research topics, or asks about events that may have occurred recently.


# Brave Web Search

Enables real-time web search via the Brave Search API.

## Instructions

Call the `run_js` tool with:
- **script name**: `index.html`
- **data**: a valid JSON string — keep it short and simple

### CRITICAL: JSON format rules

The data field MUST be a single valid JSON object. Follow these rules strictly:

- `query` must be a SINGLE string, not multiple strings
- Do NOT split the query into multiple quoted words
- Keep the query concise: 2-6 keywords maximum
- Write English keywords when possible for better results

CORRECT examples:
```
{"query":"OpenClaw history 2025","count":5}
{"query":"AI news today","count":5,"freshness":"pw"}
{"query":"比特币价格","count":5}
```

WRONG examples (never do this):
```
{"query":"OpenClaw" "2月" "火" "到"}   ← multiple strings, INVALID
{"query": "搜索最近的关于OpenClaw从2月份爆火到现在的完整经历"}  ← too long
```

### Parameters

- `query` (string, required): 2-6 keywords, keep short
- `count` (number, optional): 1-10, default 5
- `freshness` (string, optional): `"pd"` past day, `"pw"` past week, `"pm"` past month

### For long research tasks

If the user wants a long report, call the tool **multiple times** with different queries, then synthesize all results into the final answer. Example for "OpenClaw发展历史":
1. `{"query":"OpenClaw 2025 launch","count":5}`
2. `{"query":"OpenClaw viral February 2025","count":5}`
3. `{"query":"OpenClaw latest news","count":5,"freshness":"pm"}`

## Interpreting Results

- `result.web_results`: array of `{title, url, description, age}`
- `result.news_results`: array of `{title, url, description, age, source}`
- `result._debug`: which proxy succeeded

Summarize results in the user's language. Cite sources with URLs.

---
> Source: [SimonLeen22/brave-for-AI-Edge-Gallery](https://github.com/SimonLeen22/brave-for-AI-Edge-Gallery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
