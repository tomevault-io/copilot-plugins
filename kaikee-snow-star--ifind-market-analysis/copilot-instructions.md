## ifind-market-analysis

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目简介

基于同花顺 iFinD MCP 服务的 **A 股大盘综合分析工具**，一键输出完整的市场研判报告，含板块轮动追踪、资金流向分析、收盘策略与仓位建议。

## 运行命令

```bash
# 完整大盘分析（默认模式）
node src/market_analysis.js

# 9:45 早盘战情检查（仅查询 3 只目标 ETF 实时行情）
node src/market_analysis.js --945

# Windows 一键运行（带环境检测 + 时间自动判断模式）
run.bat

# Linux/Mac
bash start.sh
```

## 前置条件

- Node.js 18+
- `mcp_config.json` — iFinD MCP 认证密钥（已 gitignore，需自行创建）
- 可选：`config.json` — 飞书 Webhook URL 配置（已 gitignore）
- 可选：`npm install -g stock-sdk-mcp` — 板块深度补充分析

## 架构概览

```
                 ┌─────────────────────────────┐
                 │     market_analysis.js      │  ← 核心分析引擎 (~2320行)
                 │  入口 + 数据协调 + 报告输出  │
                 └──────┬──────────┬───────────┘
                        │          │
          ┌─────────────┘          └──────────────┐
          ▼                                        ▼
  ┌──────────────┐                        ┌────────────────┐
  │  iFinD MCP   │                        │ stock-sdk MCP  │
  │ (主数据源)    │◄──── JSON-RPC ────►   │ (补充数据源)    │
  │ call-node.js │   HTTP over HTTPS      │stock_sdk_bridge │
  │ call.py      │                        │  .js            │
  └──────┬───────┘                        └───────┬────────┘
         │                                        │
         ▼                                        ▼
  ┌──────────────┐                        ┌────────────────┐
  │  降级数据源   │                        │  HTTP 直连降级  │
  │ ashare_bridge│                        │ EastMoney/腾讯  │
  │   .js + .py  │                        │   API 兜底      │
  └──────────────┘                        └────────────────┘
```

### 模块职责

| 文件 | 职责 | 行数 |
|:---|:---|:---:|
| `src/market_analysis.js` | **核心引擎** — 数据采集→分析→报告生成→飞书推送，协调所有模块 | ~2320 |
| `src/stock_sdk_bridge.js` | **双层桥接** — 第1层 stock-sdk MCP + 第2层 HTTP API(东方财富/腾讯)降级 | ~310 |
| `src/xueqiu_data_source.js` | **雪球数据聚合层** — 板块K线回填、概念行情、申万行业→SL代码映射 | ~250 |
| `src/market_sentiment.js` | **舆情情绪分析** — 新闻关键词情感打分(正面/负面/中性词典) | ~200 |
| `src/opportunity_alerts.js` | **预警规则引擎** — 市场级/板块级/组合信号多条件检测 | ~180 |
| `src/ashare_bridge.js` | **A股备用数据源** — 封装腾讯行情API(异步fallback) | ~80 |
| `src/ashare_data.py` | **A股独立数据源** — 腾讯行情+东方财富财务，零外部依赖(stdlib+curl) | ~450 |
| `src/call-node.js` | **iFinD MCP Node.js 封装** — JSON-RPC over HTTP，零依赖 | ~160 |
| `src/call.py` | **iFinD MCP Python 封装** — 同上，备选 | ~140 |

### 数据容错层级

4 层递进，保证任意 API 不可用时仍能输出报告：

1. **iFinD MCP** — 指数/行业/概念板块行情（主数据源）
2. **雪球 HTTP API** (xueqiu_data_source) — 板块K线回填、今日涨跌幅降级补偿
3. **腾讯行情 API** (ashare_bridge) — 指数行情交叉验证
4. **东方财富 HTTP API** (stock_sdk_bridge) — 板块成分股/深度分析兜底

### 报告结构

输出包含 10 大模块的 Markdown 报告，自动保存到 `reports/` 目录：
- 指数行情 → 市场宽度 → 行业/概念板块 → 量价分析 → 综合研判
- 板块轮动追踪(RS 动量) → 量价资金流向 → 多因子评分(100分制) → 收盘策略(动态仓位锚)
- 飞书卡片推送（配置 webhook 后自动执行）

### 无测试文件

本项目为纯 Node.js 脚本，无 package.json 和测试框架。所有模块通过 `require()` 引入，无 npm 依赖。测试方式为直接运行 `node src/market_analysis.js` 观察输出。

### iFinD MCP 协议

`call-node.js` 实现 MCP (Model Context Protocol) 客户端，通过 JSON-RPC over HTTPS 与同花顺远程服务通信：
- 使用固定的 Session-Id 维持会话
- 每个请求带自增 req_id
- 支持 7 个子服务（stock/fund/edb/news/bond/global_stock/index）
- 限频规则：免费 ≤2 req/s，个人 ≤5 req/s，企业 ≤10 req/s

### stock-sdk 桥接策略

`stock_sdk_bridge.js` 通过子进程启动 stock-sdk-mcp 并读取临时文件结果（Windows stdio 缓冲问题绕开方案）：
- 60s 结果缓存避免同次运行重复请求
- 首次调用自动缓存工具列表
- MCP 进程每次调用独立启动+退出

### 关键配置

- **3 只目标 ETF**：半导体ETF(512480)、创新药ETF(159992)、港股通互联网ETF(159792) — 9:45 早盘检查标的
- **板块覆盖**：20 个申万一级行业 + 每日动态筛选 TOP10 概念板块
- **策略框架**：板块相对强度(RS)、动态仓位锚、华西情绪分析、五步复盘法

---
> Source: [kaikee-snow-star/iFind-Market-Analysis](https://github.com/kaikee-snow-star/iFind-Market-Analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
