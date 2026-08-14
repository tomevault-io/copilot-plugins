## ale-quant

> > 本文件为所有 AI agent 与贡献者的强制约束。与 [CLAUDE.MD](CLAUDE.MD) 互补：

# AGENTS.md — 项目约束文件

> 本文件为所有 AI agent 与贡献者的强制约束。与 [CLAUDE.MD](CLAUDE.MD) 互补：
> CLAUDE.MD 定义工作原则与系统设计，本文件定义功能真源、工作顺序、铁律、目录与验证。

## 一、唯一功能真源

当前功能**只依据 `docs/` 下的三份文档**：

| 真源 | 文件 |
|---|---|
| 系统总体拓扑结构 | [docs/系统总体拓扑结构.md](docs/系统总体拓扑结构.md) |
| 策略数学引擎 | [docs/策略数学引擎.md](docs/策略数学引擎.md) |
| 进化计算引擎 | [docs/进化计算引擎.md](docs/进化计算引擎.md) |

**三份文档没有定义的功能，不进入实现。** 遇到文档未覆盖的需求，先停下，回到文档或与维护者确认，不得自行臆测扩展。

## 二、工作顺序

1. **涉及策略与回测**：先读对应文档（`docs/量化交易平仓策略.md`、`docs/进化文档.md`）再动手。
2. **涉及 Go 后端**：遵守 GORM Code-First，schema 真源是 Go struct，只用 `AutoMigrate`，永不写 SQL migration 文件。
3. **涉及价格计算**：优先无量纲表达（对数收益率或比率），禁止跨标的比较绝对价格。
4. **涉及架构边界**：保持 SaaS-Strategy-Agent 分工，不做预防性解耦。

## 三、核心约束（五条铁律）

1. **复利前置**：策略必须满足复利前置条件。
2. **策略同构**：回测与实盘调用**同一个** `Step()` 实现，内部禁止 `if isBacktest` 分支。
3. **执行边界**：`Step()` **只在 SaaS 侧执行**。
4. **策略纯函数**：策略包内部禁止网络请求、数据库读写、任何文件 I/O。
5. **凭证隔离**：API Key 只能存在于 `config.agent.yaml`，永不进入 SaaS 侧。

## 四、代码目录

| 目录 | 职责 |
|---|---|
| `cmd/saas/` | SaaS 云端入口。决策大脑，执行 `Step()` 并下发交易指令；不持有 API Key。 |
| `cmd/agent/` | Agent 本地入口。执行手，调用交易所下单并上报结果；不含策略代码。 |
| `internal/saas/` | SaaS 侧业务逻辑：决策编排、API、调度。`Step()` 只在此侧执行。 |
| `internal/agent/` | Agent 侧业务逻辑：下单执行与结果上报；凭证仅来自 `config.agent.yaml`。 |
| `internal/strategy/` | 策略框架：`Step()` 契约、策略注册、基因空间与评估接口。 |
| `internal/strategies/[策略名]/` | 具体策略实现，各占一个子目录，实现 `strategy` 契约。 |
| `internal/quant/` | 量化计算原语纯函数：无量纲指标、Sigmoid、MAV、对数收益率等。 |
| `internal/adapters/backtest/` | 回测适配器：以历史数据驱动与实盘相同的 `Step()`；不下发真实交易。 |

## 五、验证命令

```bash
go list ./...   # 列出所有包，确认目录结构可被构建系统识别
go test ./...   # 运行全部测试
```

---
> Source: [yijuichen/ale-quant](https://github.com/yijuichen/ale-quant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
