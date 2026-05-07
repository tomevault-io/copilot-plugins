## gongzhonghao-agent-team

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Media Agents** automation system that uses Claude Code to automatically generate and publish articles to WeChat Official Accounts (公众号). Each subdirectory represents a different content vertical/topic area with its own automation scripts and Claude agents.

## Architecture

### Core Pattern

Each vertical follows the same architecture:
1. **Python automation script** (`daily_*.py`) - Orchestrates the workflow
2. **Claude agents** (`.claude/agents/*.md`) - Define specialized AI behaviors for content generation
3. **MCP servers** (`.mcp.json`) - Connect to external tools (WeChat publishing, Playwright for web scraping, arXiv for papers)

### Verticals

| Directory | Topic | Key Script |
|-----------|-------|------------|
| `media_agent/` | AI/Tech News | `daily_ai_news.py` |
| `digital_nomad/` | Digital Nomad/Solopreneur | `daily_nomad_news.py` |
| `investment_insights/` | Investment/Finance | `daily_investment_news.py` |
| `fuson/` | Fashion (Knitwear) | `daily_fashion.py` |
| `intelligent_unit/` | arXiv Paper Analysis | `daily_arxiv_papers.py` |
| `aml_compliance/` | AML/Sanctions Compliance | `daily_aml_news.py` |
| `military_frontier/` | Military/Defense | `daily_military_frontier.py` |
| `digital_tech/` | Digital Products/Reviews | `daily_digital_tech_news.py` |

### How It Works

1. Python script generates a dynamic prompt with current Beijing time and topic-specific search parameters
2. Script invokes Claude Code in headless mode (`claude -p <prompt> --permission-mode bypassPermissions`)
3. Claude uses MCP tools to:
   - Search news via Playwright (Google News with time filters)
   - Fetch detailed content from authoritative sources
   - Download images using curl
   - Write articles in Markdown format
   - Publish to WeChat via `wenyan-mcp` server
4. Articles are saved to `<vertical>/articles/` with naming pattern `YYYYMMDD_HHMM_topic_keywords.md`

## Common Commands

### Run immediate article generation
```bash
# Generate 1 article immediately with verbose output
python <vertical>/daily_*.py --now --count 1 --verbose

# Examples:
python media_agent/daily_ai_news.py --now --count 3 -v
python investment_insights/daily_investment_news.py --now --count 1
```

### Run scheduled generation
```bash
# Schedule at specific Beijing times
python <vertical>/daily_*.py --time 08:00 20:00 --count 5

# Run as background daemon
nohup python <vertical>/daily_*.py --time 09:00 --count 5 > output.log 2>&1 &
```

### Check logs
```bash
# Logs are in <vertical>/logs/
tail -f <vertical>/logs/*.log
```

## MCP Server Configuration

Each vertical has `.mcp.json` defining available MCP servers:

- **wenyan-mcp**: WeChat publishing (articles and image messages)
- **playwright**: Headless browser for web scraping
- **arxiv** (intelligent_unit only): arXiv paper search and download

## Agent System

Agents are defined in `.claude/agents/*.md` files with YAML frontmatter:
- `name`: Agent identifier
- `description`: When to use this agent
- `model`: Which Claude model (e.g., sonnet)
- `color`: UI color coding

Key agents:
- `*-editor` agents: Write and publish articles
- `*-analyst` agents: Deep research and analysis
- `ai-tech-editor`: Specialized for image message publishing

## Key Patterns

### Time Handling
- All scripts use Beijing time (`Asia/Shanghai` timezone via `zoneinfo`)
- News freshness is critical - typically 24-hour window with `&tbs=qdr:d` Google filter
- Time verification is triple-checked before article generation

### Article Format
- Markdown with YAML frontmatter (`title`, `cover`)
- 1500-2500 words depending on vertical
- 3-8 images minimum, downloaded via curl (not screenshots)
- Uses `agentera-*` themes for WeChat styling

### Deduplication
- Check existing articles via `Glob` on `articles/*.md`
- Extract topics from filenames to avoid repetition
- Cross-reference across sources for validation

## Dependencies

Python packages (commonly used):
- `schedule`: Task scheduling
- `zoneinfo`: Timezone handling
- `argparse`: CLI argument parsing
- `subprocess`: Claude Code invocation

---
> Source: [floodsung/gongzhonghao_agent_team](https://github.com/floodsung/gongzhonghao_agent_team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
