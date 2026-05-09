## visualize-data

> Implement CLI and web-based data visualization for `myquery`, allowing users to visualize query results (e.g., sales trends, category comparisons, etc.) directly after execution.

# 🧩 Task: Data Visualization System for myquery

## 🎯 Goal
Implement CLI and web-based data visualization for `myquery`, allowing users to visualize query results (e.g., sales trends, category comparisons, etc.) directly after execution.

## ⚙️ Requirements
- Create a tool named `visualize_data_tool` in `tools/visualize_data_tool.py`.
- Use **Plotly** for chart generation (preferred) with fallback to **Matplotlib**.
- CLI mode: render quick ASCII preview using **Rich** or show static Plotly output in terminal-compatible mode.
- Web mode: generate interactive charts and serve via `/api/visualize` FastAPI endpoint.
- Support chart types:
  - `bar`, `line`, `scatter`, `pie`, `heatmap`
- Auto-detect chart type from data (categorical → bar/pie, numeric/time → line/scatter).

## 🧠 Steps
1. Define LangChain tool `visualize_data_tool`.
2. Input: query results (list[dict]) + metadata (user prompt, chart type).
3. Output: plot object + CLI preview.
4. If running via Web UI, return Plotly JSON for rendering.
5. Add a `--visualize` flag to CLI commands for automatic visualization.
6. Integrate visualization step after `execute_query_tool` in the main agent.

## 🧩 Cursor Hints
- Use `@cursor: define-tool visualize_data_tool`
- Keep functions pure and testable.
- Store plots temporarily in `/tmp/myquery/plots/`.
- Add support for exporting charts as `.png` or `.html`.

---
> Source: [zakirkun/myquery](https://github.com/zakirkun/myquery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
