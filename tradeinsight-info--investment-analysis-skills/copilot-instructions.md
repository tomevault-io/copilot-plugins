## investment-analysis-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin** providing investment analysis capabilities. It packages skills and agents for:

- **Fundamental analysis** — financial statements, valuation metrics, earnings, balance sheets
- **Technical analysis** — price patterns, indicators (RSI, MACD, moving averages), chart analysis
- **Sentiment analysis** — news sentiment, social media signals, market mood indicators

The plugin is designed for the Claude Code plugin marketplace.

## Plugin Architecture

This project follows the Claude Code plugin structure:

```
plugin.json              # Plugin manifest (name, description, components)
skills/                  # Skills (auto-triggered or user-invocable via slash commands)
agents/                  # Subagents (autonomous task handlers)
hooks/                   # Event hooks (PreToolUse, PostToolUse, etc.)
```

- `plugin.json` registers all components and defines plugin metadata
- Skills use YAML frontmatter (`name`, `description`, `type`) and markdown body; user-invocable skills serve as slash commands
- Agents use YAML frontmatter with `description`, `tools`, and optional `model` fields

## Key Conventions

- Skill descriptions must be highly specific trigger phrases — they determine when Claude auto-invokes the skill
- Agent `description` fields should include example user messages that would trigger them
- Use `${CLAUDE_PLUGIN_ROOT}` in hooks and skills to reference plugin-relative paths
- MCP server integrations (if any) go in `.mcp.json` at the plugin root

## Domain Context

- Financial data may come from external APIs (e.g., Yahoo Finance, Alpha Vantage, SEC EDGAR)
- Analysis outputs should include confidence levels and data source citations
- Investment analysis is informational only — always include appropriate disclaimers

---
> Source: [TradeInsight-Info/investment-analysis-skills](https://github.com/TradeInsight-Info/investment-analysis-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
