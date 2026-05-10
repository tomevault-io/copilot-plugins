## mcp-tool-definition

> >

## MCP Tool Definitions

When adding or editing an MCP Server's tool, follow this structure to make it clear when and how the tool should be used:

### Tool Title
The title should clearly indicate what the tool accomplishes or produces. Focus on the outcome or capability - what will it do for the user?
Examples:
- "Screen Stocks by Technical Patterns" (not "Stock Screening Tool")
- "Analyze Insider Trading Activity" (not "Insider Data Tool")
- "Calculate Financial Health Score" (not "Health Score Generator")

### Tool Description
The description should cover four key aspects:
1. **What is it?** - Brief explanation of the tool's purpose
2. **What does it do?** - Functionality explanation  
3. **When to use it?** - Clear use cases and scenarios
4. **What it returns/effects?** - Expected outputs or side effects

Descriptions should be comprehensive enough for an AI agent to understand when this tool is the appropriate choice over others.

Example format:
```
"This tool [what it is] that [what it does]. Use this when [when to use it, specific scenarios]. 
The tool will [what it returns/effects]. [Additional context or limitations if relevant]."
```

### Trading-Specific Examples:

**Good:**
```
"Comprehensive stock screening using Finviz filters that supports technical patterns, fundamental criteria, and multi-parameter filtering. Use this when you need to find stocks matching specific investment criteria like channel down patterns, profitable companies, or large-cap stocks. The tool returns a ranked list of stocks with key metrics including market cap, P/E ratios, and current prices. Supports complex filter combinations for advanced screening strategies."
```

**Poor:**
```
"Advanced stock filter for custom screening with multiple filter combinations."
```

**Good:**
```
"Financial health analysis tool that calculates a comprehensive score based on profitability, liquidity, leverage, efficiency, and growth metrics. Use this when evaluating the overall financial strength of a company for investment decisions or risk assessment. Returns a weighted health score (0-100) with detailed breakdowns of each category and actionable insights. Allows custom weighting of different financial factors."
```

**Poor:**
```
"Calculate financial health score for stocks."
``` 

---
> Source: [netanelavr/trading-mcp](https://github.com/netanelavr/trading-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
