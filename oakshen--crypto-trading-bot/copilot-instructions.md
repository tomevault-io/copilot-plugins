## crypto-trading-bot

> - 基于 Go 的自动加密货币交易智能体，结合 LLM、技术指标与币安期货执行形成端到端闭环

# AGENTS.md

## 项目使命
- 基于 Go 的自动加密货币交易智能体，结合 LLM、技术指标与币安期货执行形成端到端闭环
- 通过 Cloudwego Eino Graph 管理多智能体并发（分析→决策→执行），以最少人工干预实现稳定策略迭代

## 最新亮点
- 外部 Prompt 管理：`prompts/` 目录可热更新策略文案，通过 `TRADER_PROMPT_PATH` 切换不同交易风格
- 动态杠杆：`BINANCE_LEVERAGE=low-high` 允许 LLM 依据置信度/趋势/波动在区间内自适应，兼顾收益与风险

## 多智能体工作流
- 并行阶段：`market_analyst` 负责 OHLCV 与技术指标；`sentiment_analyst` 从 CryptoOracle 拉取情绪
- 顺序阶段：`crypto_analyst` 汇总币种专属数据，随后 `position_info` 查询币安持仓
- 决策阶段：`trader` 等待所有上下游数据，优先调用 OpenAI，失败时回落到内置规则；图定义位于 `internal/agents/graph.go`

## 核心模块速览
- `internal/agents/`：图定义、LLM 工具、决策逻辑
- `internal/dataflows/`：市场与情绪数据，含 RSI/MACD/BB/SMA/EMA/ATR 等指标计算
- `internal/executors/`：币安期货下单，支持单向/双向持仓、BUY/SELL/CLOSE_* 动作，含指数退避重试
- `internal/storage/`：SQLite `trading_sessions`，记录所有报告与执行结果
- 其他：`internal/config` 负责 Viper + .env；`internal/scheduler` 对齐 K 线节奏；`internal/web` 提供 Hertz 监控端口

## 运行 & 调试
```bash
make build        # 编译主机器人
make run          # 单次执行
make run-web      # 循环执行 + Web 控制台
make query ARGS="latest 5"  # 查询最近会话
make test / make test-cover  # 测试与覆盖率
```
- 调试图时设置 `DEBUG_MODE=true`，观察日志链路并可用 `make query ARGS="latest 1"` 检查数据库

## 配置要点
- `.env` 中集中管理，参考 `env.example.yaml`；修改 `internal/config/config.go` 记得同步 `.env.example`
- 关键开关：`BINANCE_TEST_MODE`（测试 vs 实盘）、`BINANCE_POSITION_MODE`（oneway/hedge）、`OPENAI_API_KEY`、`CRYPTO_TIMEFRAME`、`TRADER_PROMPT_PATH`
- 杠杆、仓位、止损等需结合 `MAX_DRAWDOWN`、`RISK_PER_TRADE` 做统一风险设定

## 测试策略
- 单元测试集中在 config/storage/scheduler/dataflows/executors 包
- 集成/币安测试仅在 `BINANCE_TEST_MODE=true` 下运行，避免真实成交

## 风险提示
- `BINANCE_TEST_MODE=false` 即为实盘，务必先小仓+低杠杆验证
- LLM 失败会自动退回规则策略，但仍需人工审查关键交易
- 日志、数据库与 Web 控制台是追踪异常的三大信号来源

## 协作备注
- 若代码注释写英文，需紧跟一行中文翻译
- Prompt 或配置变更请更新 `prompts/README.md` 与 `.env.example`
- 请用中文回答用户的问题

---
> Source: [Oakshen/crypto-trading-bot](https://github.com/Oakshen/crypto-trading-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
