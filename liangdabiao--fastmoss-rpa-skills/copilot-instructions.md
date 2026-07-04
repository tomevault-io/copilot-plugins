## fastmoss-rpa-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Purpose

`fastmoss-rpa` is a browser-RPA + data-analysis project on **fastmoss.com** (TikTok Shop analytics). All operational knowledge lives in **section-specific** bundled skills under `.claude/skills/`:

| Skill | FastMoss section |
|---|---|
| **fastmoss-products** (installed) | 商品 — 新品榜/销量榜/热推榜/视频商品榜 + single-shop new-listing cadence |
| **fastmoss-creators** (installed) | 达人 — 涨粉/带货/蓝V/热门/黑马达人榜 + 达人搜索入口 |
| **fastmoss-shops** (installed) | 店铺 — 销量榜/热推榜 + 店铺搜索入口 |
| **fastmoss-ads** (installed) | 广告引擎 — 标签洞察/关键词趋势/热门品类趋势 (table) + 电商广告/种草广告/广告主洞察 (card-based, not scripted) |
| **fastmoss-creatives** (installed) | 视频&素材 — 热门视频/热门音乐/热门标签 (table) + AI带货视频榜 (card-based, not scripted) |
| **fastmoss-livestreams** (installed) | 直播 — TT直播榜/直播爆品榜/直播带货达人榜 + 直播搜索入口 |
| **fastmoss-market** (installed) | 品类大盘 — 行业格局 (categoryDistribution API) + 市场总览 (base/salesChart APIs). API-first skill — uses page-context fetch(), not DOM scraping. |

## Authoritative reference

**Read `.claude/skills/fastmoss-products/SKILL.md` first** for any product-ranking work. It is self-contained and covers:

- Scraper scripts (`scripts/fastmoss_scraper.py`, `fastmoss_filtered.py`, `shop_scraper.py`)
- Analysis aggregator (`scripts/analyze.py` — reproduces `analysis.md` from CSVs)
- Environment quirks (`references/environment.md` — Windows/bash/jq/heredoc rules)
- Analysis report template (`references/analysis_recipe.md`)
- Gotchas: stale @e refs, heredoc+regex breakage, SPA hydration lag, no-jq, GBK-vs-UTF8

## Quick start

```bash
# Verify Kimi WebBridge daemon is healthy
~/.kimi-webbridge/bin/kimi-webbridge status

# Top 50 → CSV
python .claude/skills/fastmoss-products/scripts/fastmoss_scraper.py --pages 5 --out data/top50.csv

# Reproduce analysis.md
python .claude/skills/fastmoss-products/scripts/analyze.py \
    --top50 data/top50.csv --by-country data/by_country.csv \
    --by-category data/by_category.csv --shop data/shop_*.csv \
    --out-md analysis.md
```

## Working in this repo

Windows + bash shell. Use forward-slash paths and `/dev/null` (not `NUL`). No build/test/lint commands — outputs land in `data/`, the report lands in `analysis.md`.

---
> Source: [liangdabiao/fastmoss-rpa-skills](https://github.com/liangdabiao/fastmoss-rpa-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
