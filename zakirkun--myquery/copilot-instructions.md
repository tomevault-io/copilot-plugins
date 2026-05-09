## analyze-data

> Generate AI-based insights and summaries from query results using OpenAI models (via LangChain).

# 🔍 Task: Data Analysis Tool

## 🎯 Goal
Generate AI-based insights and summaries from query results using OpenAI models (via LangChain).

## ⚙️ Requirements
- Create `tools/analyze_data_tool.py`.
- Input: query results (list of dicts), prompt context.
- Output: textual summary or analysis.
- Example: “The average order value increased by 12% from last month.”
- Use GPT model (e.g., gpt-4-turbo or gpt-4o-mini).
- Integrate with CLI and Web UI.

## 🧠 Steps
1. Define LangChain tool `analyze_data_tool`.
2. Accept result set, user prompt, and schema context.
3. Summarize trends, patterns, or anomalies.
4. Provide both short summary and extended explanation.
5. Support output as Markdown (for Web UI rendering).

## 🧩 Cursor Hints
- Use `@cursor: define-tool analyze_data_tool`
- Keep analysis concise but informative.
- Integrate directly after `execute_query_tool` in main agent.

---
> Source: [zakirkun/myquery](https://github.com/zakirkun/myquery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
