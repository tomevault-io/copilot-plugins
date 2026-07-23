## search-console-mcp

> Optimized prompts for analyzing SEO in Claude.


Claude is excellent at synthesizing the deterministic data from our MCP server into deep strategic insights.

## The "SEO Audit" Prompt
Use this to get a comprehensive view of your site's health.

```markdown
Run a comprehensive SEO audit for https://example.com. 
1. Use the recommendations tool to find high-level opportunities.
2. Check for any keyword cannibalization.
3. Compare the last 28 days to the previous period and summarize the biggest winners and losers.
4. Provide a prioritized list of 3 actions I should take today to increase traffic.
```

## The "Traffic Drop Detective" Prompt
Use this when you notice a decline.

```markdown
My traffic for https://example.com has dropped recently. 
1. Use the time-series tool to find the exact date the drop started.
2. Tell me which pages and queries saw the biggest decrease in clicks.
3. Check the PageSpeed and Indexing status of those pages.
4. Act as a senior SEO consultant and attribute this drop to a specific cause (e.g., technical, algorithm, or seasonality).
```

## The "Site Health Check" Prompt
Use this to get an instant diagnostics overview.

```markdown
Run a health check across all my verified sites.
1. Use the site health check tool to get the status of each property.
2. For any site with "critical" or "warning" status, investigate the root cause.
3. If there was a traffic drop, check if it correlates with a known Google algorithm update.
4. Provide a prioritized action plan for the most affected sites.
```

## The "Google vs. Bing" Prompt
Use this to find cross-engine opportunities.

```markdown
Compare my performance on Google and Bing for https://example.com over the last 30 days.
1. Use the compare_engines tool to see which keywords rank well on Google but poorly on Bing.
2. Flag any keywords where I'm overly dependent on Google (85%+ click share).
3. Identify Bing opportunities where I rank in the top 5 on Google but 10+ on Bing.
4. Give me a prioritized list of 3 quick wins to grow my Bing traffic.
```

## Why these work well
*   **Chained Tool Use:** These prompts encourage Claude to use multiple tools in sequence before answering.
*   **Role Prompting:** By asking Claude to "Act as a senior SEO consultant," you improve the quality of the qualitative reasoning it adds to the quantitative data.
*   **Structured Outputs:** Requesting a "prioritized list" forces the model to weigh different findings against each other.

---
> Source: [saurabhsharma2u/search-console-mcp](https://github.com/saurabhsharma2u/search-console-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
