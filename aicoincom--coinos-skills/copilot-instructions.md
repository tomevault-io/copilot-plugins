## coinos-skills

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

CoinOS Skills — 给 AI coding 助手 (Claude Code / Hermes / OpenClaw) 用的加密 / 量化 / 链上 skill 集合, 由 AiCoin Open API + OKX Web3 DEX API 驱动.

CoinClaw 三引擎实例容器在 build 时把这个仓库 git clone 进去 (`image-*/Dockerfile` 都有 `git clone --depth 1 https://github.com/aicoincom/coinos-skills.git`), 之后通过 init-workspace 同步到用户 PVC. 改 upstream 后必须重打三个引擎镜像才生效, 见 [reference_coinos_skills_pipeline](../coinclaw/...) (memory).

## Architecture

```
coinos-skills/
├── skills/
│   ├── aicoin-market/      # Prices, K-lines, news, signals, airdrops
│   ├── aicoin-account/     # Balance, positions, API key, registration
│   ├── aicoin-trading/     # CEX trading (Binance/OKX 等)
│   ├── aicoin-freqtrade/   # 策略 / 回测 / 部署 / 实时控制
│   │   ├── lib/
│   │   │   ├── coinclaw-env.mjs       # 三引擎自动识别
│   │   │   ├── freqtrade-api.mjs      # REST + auth
│   │   │   ├── strategy-builder.mjs   # IStrategy 代码生成
│   │   │   ├── aicoin_data.py         # 给 IStrategy 用的 Python SDK
│   │   │   └── defaults.json          # 内置免费 key
│   │   ├── scripts/
│   │   │   ├── ft.mjs            # daemon 控制 (REST + 配置变更)
│   │   │   ├── ft-deploy.mjs     # 策略生命周期 + backtest/hyperopt
│   │   │   └── ft-dev.mjs        # 调试 (回测 / 蜡烛 / 策略详情)
│   │   └── strategies/           # 模板策略 (FundingRate / Whale / Liq)
│   ├── aicoin-hyperliquid/ # HL 鲸鱼 / 清算 / OI / trader analytics
│   └── aicoin-onchain/     # 链上 DEX swap / 钱包 / gas
├── AGENTS.md               # Skill 路由 (给 agent 看)
├── CLAUDE.md               # 开发说明 (本文档)
└── .claude-plugin/         # 插件 metadata
```

每个 skill 自包含 (`SKILL.md`, `lib/`, `scripts/`). aicoin-* skill 共用 `../lib/aicoin-api.mjs`; aicoin-onchain 用自己的 `lib/okx-api.mjs`.

## CoinClaw 三引擎适配

`aicoin-freqtrade` 是改造重点 — `lib/coinclaw-env.mjs` 通过 sentinel 文件 (`/workspace/.hermes` / `.claude` / `/home/node/.openclaw`) 自动识别引擎, 解析对应的:

- `freqtradeUserdir` (`--userdir`)
- `strategyPath` (`--strategy-path`, daemon 真读)
- `configPath` (`config.json`)
- `envFile` (`.env` 真路径)
- `ftPassFile` (`.ft_api_pass`)
- `supervisorSocket` (supervisord RPC unix socket)

`scripts/ft-deploy.mjs` `scripts/ft.mjs` 两个 CLI 自动选择路径, 不需要用户配 `FREQTRADE_*` 环境变量. 默认 user 改成 `freqtrade` (修了历史 bug — 之前默认 `freqtrader` 但容器里 daemon 真实 user 是 `freqtrade`, 401).

CoinClaw 容器外 (用户 macOS / Linux), `coinclawEnv()` 返回 `null`, ft-deploy.mjs 退到 host 模式 (git clone freqtrade + nohup). 双模式同一份 skill, 用户感知不到.

## 关键设计决定

### 1. ft-deploy.mjs 不能跟 supervisord 抢 port 8080

老版本 ft-deploy.mjs 的 `deploy` 直接 `nohup freqtrade trade & echo $! > pid`, 在容器里跟 supervisord 管的 daemon 抢端口, dashboard 立刻 offline. 改造后 coinclaw 模式下 deploy 流程是: 写策略 → 改 `config.strategy` → `restartDaemon()` (supervisorctl 优先, kill+autorestart 兜底).

### 2. 切策略需要 daemon 重启 (reload_config 不切策略)

freqtrade 1.x 的 `/api/v1/reload_config` 重新加载 config, 但**不重新加载 IStrategy 类**. 切策略必须让 freqtrade 进程退出 + supervisord 拉起 — `ft.mjs set_strategy` / `ft-deploy.mjs deploy` 都走这个流程.

切交易对 (`set_pairs`) 不需要重启, `reload_config` 即可.

切实盘 (`set_dry_run`) 必须重启 — exchange 客户端在启动时根据 `dry_run` 选 ccxt vs ccxt sandbox, 运行中切不掉.

### 3. dashboard 数字对齐 = `/api/v1/profit`

CoinClaw web UI 右侧 dashboard 的累计盈亏来自 `profit_closed_coin` (已平仓累计 USDT). 老 SKILL.md 没要求 agent 调 `/profit`, 用户问"赚多少"时 agent 只调 `/status` (open trades 浮动) 然后回答 -44.30, 但 dashboard 显示 +95.88 (closed +138 - 浮动 42), 数字不一致, 用户立刻不信任.

新 SKILL.md 写明: 回答盈亏类问题**必须先调 `ft.mjs profit`**, 同时报 `profit_closed_coin` + `profit_all_coin`, 跟 dashboard 顶栏对齐.

## 测试

新增改动后必须:

1. `cd skills/aicoin-freqtrade && node --check lib/*.mjs scripts/*.mjs` 语法检查
2. 在 host 模式 (本地) 跑 `node scripts/ft-deploy.mjs strategy_list` / `node scripts/ft.mjs ping` 看返回的 `mode` 字段是 `host` (因为本地没 freqtrade daemon)
3. 三引擎重打镜像, 起新实例, 在 chat UI 里问"现在赚多少 / 切到 RSI 策略 / 切实盘", 看 agent 是否走脚本回答 + dashboard 是否同步刷新

## 改 upstream 后的发布流程

1. 在 fork 分支提 commit
2. push 到 `aicoincom/coinos-skills` 的对应分支
3. 改三引擎 image 重打: `image/Dockerfile`, `image-hermes/Dockerfile`, `image-claude-code/Dockerfile` 都有 `git clone --depth 1 ... coinos-skills.git` — Kaniko 没 layer cache, 每次 build 都 clone main
4. 用户 pod 下次重启时 init-workspace 把新 skills 同步到 PVC

热补丁: `kubectl exec` 进 pod 直接改 `/workspace/.hermes/skills/aicoin-freqtrade/scripts/ft.mjs`, 但只活到下次 init-workspace — 重启后被 image template 覆盖.

---
> Source: [aicoincom/coinos-skills](https://github.com/aicoincom/coinos-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
