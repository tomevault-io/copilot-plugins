## edge-gallery-parallel

> Search the web using the Parallel AI search engine to find relevant, high-quality results for any topic, question, or research query.


# Parallel AI Search

## Instructions

Call the `run_js` tool using `index.html` and a JSON string for `data` with the following fields:
- **query**: Required. The search query string. Formulate a clear, specific search query based on the user's request. Use natural language — Parallel AI works best with descriptive queries rather than keyword-style queries.
- **numResults**: Optional. Number of results to return (1–10). Defaults to 5.

**Constraints:**
- Summarize the search results in a clear, well-structured response using the SAME language as the user's original prompt.
- Include the title, URL, and a brief description for each result so the user can evaluate relevance.
- If highlights or summaries are available in the results, use them to provide richer answers.
- For time-sensitive queries, mention that results may vary based on Parallel AI's index freshness.

---
> Source: [megakilo/edge-gallery-parallel](https://github.com/megakilo/edge-gallery-parallel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
