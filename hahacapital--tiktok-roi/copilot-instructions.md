## tiktok-roi

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project

**TikTok ROI 智能运营平台** — AI-driven operations tool for TikTok Japan cross-border e-commerce, centered on ROI (break-even ≥2.5) as the core decision metric.

Currently **docs-only**: no application code yet. Planning, contracts, and research live in `docs/`.

## Repository Layout

- `docs/README.md` — project overview, team, roadmap, tech stack
- `docs/TODO.md` — phased task list with owners/deadlines
- `docs/MVP产品设计.md` — MVP spec: "ROI 定价决策工作台" web app with 7 features (F1–F7)
- `docs/MVP成本估算.md` — cost estimate (dev + data platforms + cloud + API)
- `docs/数据平台调研报告.md` — data source research (FastMoss, TikTok APIs, 1688 via Onebound/Pangolin)
- `docs/会议纪要-*.md` — meeting notes on workflow and AI改造 plans
- `docs/尧的视频会议.txt` — raw meeting transcript
- `docs/软件开发服务合同.md` — dev services contract
- `docs/scripts/sync_feishu.sh` — Feishu wiki sync via `lark-cli` (requires prior `lark-cli auth login`)

## MVP Scope (Phase 1, 18 days)

Single web app. Seven features only — stay within scope:

| # | Feature | Days |
|---|---|---|
| F1 | 运费计算器 | 2 |
| F2 | ROI 计算器 & 盈亏模拟 | 3 |
| F3 | 竞品价格带分析 (FastMoss) | 3 |
| F4 | LLM 智能定价建议 (Claude Sonnet 4.6) | 3 |
| F5 | 商品工作台 / 决策存档 (MVP 灵魂) | 3 |
| F6 | 店铺自测 Dashboard (TikTok Partner API) | 2 |
| F7 | 1688 图搜反查货源 (Onebound) | 1 |

**Out of MVP scope** (do NOT expand into these without explicit approval): 达人评估, 投流优化, 内容生成, 浏览器插件, 订单分拣, 活动提醒, 多店铺.

## Planned Tech Stack

- Frontend: Web app (no browser extension in MVP)
- Data: FastMoss OpenAPI, TikTok Shop Partner API v2, TikTok Ads API, EchoTik, Kalodata, Onebound (primary) + Pangolin (failover) for 1688
- AI: Claude Sonnet 4.6 for pricing suggestions, title/description generation

## Team

- 赵总 (owner) · 李远哲 (BD) · 孙栩尧 (ops lead) · 张翼翔 (dev)

## Working Notes

- Docs are primarily in Chinese; preserve Chinese terminology when editing.
- When implementation begins, create source directories alongside `docs/` (e.g. `apps/`, `packages/`) and update this file with build/test/lint commands.
- Contract pricing and MVP timelines in `docs/软件开发服务合同.md` are authoritative for scope negotiations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hahacapital)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hahacapital)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
