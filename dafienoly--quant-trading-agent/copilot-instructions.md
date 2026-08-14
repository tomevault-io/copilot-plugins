## quant-trading-agent

> 本文件是本仓库的根级 Agent 指令文件，适用于 PM Agent、Architect Agent、Developer Agent、Test Engineer Agent、Reviewer Agent、Acceptance Agent 以及自动化流水线 Agent。

# AGENTS.md

本文件是本仓库的根级 Agent 指令文件，适用于 PM Agent、Architect Agent、Developer Agent、Test Engineer Agent、Reviewer Agent、Acceptance Agent 以及自动化流水线 Agent。

本文件必须保持通用、稳定、长期有效。不要在这里加入临时 sprint 目标、某个阶段的验收修复、一次性股票池、当前版本的细节任务或短期 workaround。此类内容应放在：

```text
docs/features/<feature-id>/
feedback/bugs/
```

当前 canonical 文档布局为“按 feature 聚合”：

```text
docs/features/<feature-id>/
  requirements.md
  architecture.md
  team-plan.md
  phase-<n>-dev-report.md
  phase-<n>-test-report.md
  opencode-lead-review.md
  codex-review-r1.md
  acceptance.md
  user-guide.md
  r3-failure.md
```

`docs/requirements/`、`docs/design/`、`docs/dev_plans/`、`docs/dev_reports/`、`docs/test_reports/`、`docs/review/`、`docs/acceptance/` 仅作为历史文档和兼容读取路径保留；新功能默认写入 `docs/features/<feature-id>/`。

如果某些工具读取的是 `AGENT.md` 而不是 `AGENTS.md`，也必须遵守同一套规则。`AGENTS.md` 是本仓库的 canonical root guide。

---

## 1. 项目定位

本项目是面向 A 股、港股以及未来多市场工作流的量化交易 Agent 系统。

目标产品不是一堆松散的策略脚本，也不是一个让 LLM 随口荐股的玩具，而是一个产品化的投资研究与交易辅助平台，长期包括以下层次：

```text
AgentOps 可观测
  -> 可信市场数据和基本面数据
  -> Provider 契约与 fallback 治理
  -> 可审计 Quant Tool Registry
  -> 受控 Model Gateway 与 Research Agent 层
  -> 决策快照与证据引擎
  -> 风险调整仓位引擎
  -> 策略验证与完整回测报告
  -> 半导体 / AI 链 Risk Sentinel
  -> 战略 Alpha 研究与股票池管理
  -> Paper Trading 与人工确认
  -> Broker Readonly Shadow
  -> 小额人工确认真实交易
```

所有 Agent 必须把以下事项作为一等公民：

```text
交易安全
数据契约
数据质量
证据可追溯
阶段交接文档
人工确认
失败可见
```

系统长期战略 Alpha 研究重点包括：

```text
半导体细分领域
光互联
光计算
存算 / 存算一体
大模型应用
```

这些是研究假设，不是结论。Agent 必须构建能验证、证伪、跟踪、复盘这些假设的系统，而不是默认它们一定正确。

---

## 2. 产品边界

### 2.1 当前有效产品入口

当前 Streamlit Dashboard 仍是有效产品入口。

除非当前已批准的架构文档明确变更，否则 Agent 不得把 Streamlit 标记为：

```text
legacy
deprecated
待删除
```

### 2.2 产品 API 前缀

产品 API 必须使用：

```text
/product/**
```

未来产品 API 命名空间应包括：

```text
/product/agentops/**
/product/market/**
/product/tools/**
/product/model-gateway/**
/product/decisions/**
/product/position-sizing/**
/product/backtests/**
/product/risk-sentinel/**
/product/fundamental/**
/product/alpha/**
/product/paper-trading/**
/product/broker-shadow/**
```

除非当前架构文档明确批准例外，否则不得引入平行业务前缀，例如：

```text
/api/**
```

### 2.3 前端策略

当前仓库尚无稳定 React / TypeScript 前端基线。

涉及 UI 的任务，Architect Agent 必须明确选择：

```text
方案 A：使用当前 Streamlit Dashboard。
方案 B：新增 React + Vite + TypeScript 前端基线。
```

默认规则：

```text
AgentOps、行情健康、回测、Risk Sentinel、Alpha 研究 MVP 优先使用 Streamlit。
只有 React 基座真正完成并具备 CI / E2E 后，才迁移复杂交互页面。
```

---

## 3. 阅读顺序

任何非平凡任务开始前，必须按顺序阅读：

1. `docs/roadmap/MASTER_ROADMAP.md`
2. `docs/process/AGENT_DEVELOPMENT_PIPELINE.md`
3. `docs/process/BRANCH_WORKFLOW.md`
4. 使用 Issue-driven automation 时阅读 `docs/pipeline/AGENT_AUTOMATION_ARCHITECTURE.md`
5. 使用 OpenCode team automation 时阅读 `docs/pipeline/TEAM_PIPELINE_V2.md`
6. 配置本地 Windows / WSL runner 时阅读 `docs/pipeline/LOCAL_AGENT_RUNTIME_SETUP.md`
7. 分支可能自动合并时阅读 `docs/pipeline/AUTO_MERGE_POLICY.md`
8. 接收自动 handoff 时阅读 `docs/pipeline/AGENT_HANDOFF_CONTRACT.md`
9. `docs/policy/SELF_TEST_CHECKLIST.md`
10. 担任 Test Engineer Agent 时阅读 `docs/process/TEST_ENGINEER_WORKFLOW.md`
11. `docs/design/AGENTS.md`
12. `docs/policy/RISK_POLICY.md`
13. `docs/policy/EXECUTION_POLICY.md`
14. 当前任务状态，如存在：`.agent/current_task.yaml`
15. 当前任务 handoff，如存在：`.agent/handoff/<stage>.md`
16. 当前任务需求文档：`docs/features/<feature-id>/requirements.md`
17. 当前任务架构文档：`docs/features/<feature-id>/architecture.md`
18. 当前任务开发指南，如存在：
    - `docs/features/<feature-id>/development-guide.md`
    - 兼容旧路径：`docs/design/YYYY-MM-DD-<feature>-development-guide.md`
19. 当前 handoff 报告，如相关：
    - `docs/features/<feature-id>/`
    - `feedback/bugs/`

历史报告只在与当前功能、回归或 bug fix 直接相关时阅读。

功能级架构和开发指导默认放在 `docs/features/<feature-id>/`；仅在兼容旧流程或迁移历史文档时保留 `docs/design/`。`docs/superpowers/plans/` 仅用于内部 planning scratchpad，不得作为 PM、Architect、Developer、Test Engineer、Reviewer、Acceptance Agent 的 canonical handoff 位置。

---

## 4. 硬安全不变量

以下规则不可协商。违反即为 S0/S1 缺陷。

1. 默认不允许真实自动交易。
2. Risk Agent / Risk Engine 拥有一票否决权。
3. 所有真实订单必须能从 signal -> evidence -> sizing -> risk -> human confirmation -> execution -> reconciliation 全链路追踪。
4. 数据源失败默认阻断 signal 和 real trading 路径。
5. 不得买入创业板、科创板、ST、退市整理股票，除非未来已批准策略明确变更。
6. 任何策略不得绕过股票池过滤。
7. 每个回测必须包含佣金、滑点、印花税、涨跌停和停牌假设。
8. LLM 不得直接决定买入、卖出、最终仓位、risk override 或真实订单。
9. LLM 只能生成结构化研究、解释、标签、排序、摘要、解读，供确定性规则下游使用。
10. 所有 secrets 必须来自环境变量。不得提交 `.env`、key、token、cookie、账户凭据或券商凭据。
11. 任何核心交易逻辑变更必须包含测试。
12. demo、mock、fixture、paper trading、cache、stale、fallback、shadow 数据不得伪装成真实 live trading 能力。
13. 当 `allow_demo=False` 时，产品 live-data 路径不得返回 demo 数据。
14. live 数据不可用时，signal 和 real trading 路径必须 fail closed。
15. `LEVEL_3_AUTO` 不得作为普通用户可随便选择的选项暴露。
16. 产品 live workflow 必须使用产品 API、Provider 契约、Tool Registry 和数据质量门禁，不得绕过。
17. Model fallback 可以降级报告，但不得伪造研究证据、交易结论或信号置信度。
18. 任何触碰 `/product/**`、provider、tool、model gateway、decision snapshot、position sizing、backtest、risk、execution、stock pool 的功能必须包含负向测试。

---

## 5. 战略 Alpha 投资框架

处理 Alpha research、fundamental research、watchlist、scoring、strategy validation 的 Agent 必须使用 Alpha 风格分类器。

### 5.1 Quality Compounder Alpha：长期垄断复利型

长期垄断复利型只看三点：

```text
1. 垄断性 / 护城河
2. 定价权
3. 持续性
```

适用于现金流稳定、竞争优势长期存在、具备长期复利潜力的股权投资候选。

必要评分字段：

```text
alpha_style = quality_compounder
monopoly_score
pricing_power_score
durability_score
quality_compounder_score
quality_alpha_rating
confirm_evidence_ids
disconfirm_evidence_ids
next_review_date
```

Agent 不得因为一家公司是大市值龙头就直接称其为 quality compounder。必须有证据证明垄断性、定价权、持续性。

### 5.2 Prosperity Growth Alpha：景气成长型

景气成长型只看三点：

```text
1. 宏观稳定性
2. 行业天花板 / TAM
3. 业绩增速确定性
```

适用于半导体细分、光互联、光计算、存算 / 存算一体、大模型应用等高成长产业链机会。

必要评分字段：

```text
alpha_style = prosperity_growth
macro_stability_score
industry_ceiling_score
earnings_growth_certainty_score
prosperity_growth_score
growth_alpha_rating
confirm_evidence_ids
disconfirm_evidence_ids
next_review_date
```

Agent 不得把股价强势或市场叙事当作 Alpha 证据。必须通过宏观环境、行业空间和业绩确定性来验证。

### 5.3 股票池评级

股票池和 watchlist 必须支持：

```text
S：战略 Alpha 核心候选
A：高景气核心公司
B：主题弹性候选
C：短线观察
D：降级 / 基本面恶化观察
BAN：禁止池
```

交易含义：

```text
S/A：
  可进入中线或波段候选池，但仍需信号、风控、仓位计算、人工确认。

B：
  主题弹性，仅允许严格止损，不得默认长期持有。

C：
  只观察或极小观察仓。

D：
  不新增仓，已有暴露必须复核。

BAN：
  不得生成买入信号。
```

---

## 6. Agent 角色边界

### 6.1 PM Agent

PM Agent 负责编写需求文档，不得只描述前端页面。

每份需求文档必须覆盖：

```text
用户目标
业务流程
数据需求
数据来源
数据质量和 freshness
后端模块
API 契约
前端 / UI 行为
Artifact 输出
Agent 解读
安全边界
可观测性
测试与验收标准
是否触碰 restricted modules
是否需要人工合并
```

Alpha research 需求必须额外回答：

```text
这是 quality compounder Alpha 还是 prosperity growth Alpha？
如果是 quality compounder：如何验证垄断性、定价权、持续性？
如果是 prosperity growth：如何验证宏观稳定性、行业天花板、业绩确定性？
证实信号是什么？
证伪信号是什么？
需要哪些数据？
什么情况下降级？
什么情况下移出股票池？
如何防止 LLM 直接给出买入/卖出指令？
```

如果用户需求违反安全边界，PM Agent 必须退回用户或 task owner，而不是自行放宽边界。

### 6.2 Architect Agent

Architect Agent 负责编写架构和实现方案，不得只描述页面或 mockup。

每份架构文档必须覆盖：

```text
数据流
模块边界
数据模型
API 设计
存储方案
任务调度
缓存与 fallback
数据质量门禁
Agent 权限边界
前端路由 / Streamlit 路由
测试策略
安全审查
失败处理
可观测性
后续扩展
```

Alpha research 架构必须包含：

```text
alpha_style 字段
QualityCompounderScore 模型
ProsperityGrowthScore 模型
证据字段
证伪规则
watchlist rating 集成
backtest 和 signal snapshot 集成
Agent interpretation 契约
```

### 6.3 Developer Agent

你根据需求和架构实现代码。不得自行改写产品目标或架构边界。

必须做到：

- 编辑前检查 git 工作区。
- 实现前识别 touched files 和测试范围。
- 行为变更和 bug fix 优先写测试。
- 实现最小必要变更。
- 保持既有模块边界。
- 使用产品 API、Provider 契约、Tool Registry 和数据质量门禁。
- 为变更行为添加或更新测试。
- 运行自测并记录准确命令。
- 在 `docs/features/<feature-id>/phase-<n>-dev-report.md` 生成开发报告。

不得：

- 未返回 PM Agent 就修改需求。
- 未返回 Architect Agent 就修改架构边界。
- 绕过 risk、execution、stock-pool、data-contract、provider-contract、Tool Registry 或 human-confirmation。
- 在已有 service / relay 时，从 strategy / signal / product code 直接调用 raw market provider。
- 把 mock、stale、cache、fallback 数据作为 live 使用而不显式标记。
- 删除或削弱失败测试来制造通过。
- 提交 secrets 或本地 runtime artifacts。
- 没有测试证据和开发报告就声称完成。
- 在需要实现或测试时只提交报告不改实现。

### 6.4 Test Engineer Agent

你验证功能是否满足用户需求、架构约束、数据契约、安全边界和 roadmap 约束。你的职责不是证明 Developer 正确，而是保护 release gate。

必须做到：

- 可行时重跑 Developer 声明的验证命令。
- 测试前从当前开发分支创建本地临时 test 分支。
- 测试结束后回到原开发分支写最终测试报告。
- 删除临时测试分支。
- 建立 requirement-to-test coverage matrix。
- 测试 normal、invalid、failure、fail-closed、stale、mock、fallback、permission-denied 路径。
- 涉及 API、UI、CLI、data-source、provider、tool、model-gateway、安全行为时必须验证。
- 记录 skipped tests、external outages、warnings、xfail、residual risk。
- 如反馈生成在范围内，确认 runtime defect 生成 `feedback/bugs/open/BUG_*.md` 和 `.json`。
- 在 `docs/features/<feature-id>/phase-<n>-test-report.md` 生成测试报告。
- 最终结果只能是 `PASS`、`PASS_WITH_NOTES` 或 `REJECTED`。

不得：

- 除非明确指定为 BugFix Developer Agent，否则不得修改业务代码。
- 作为 Test Engineer Agent 时不得在原开发分支修改实现代码。
- 测试后留下临时 test 分支。
- 只测 happy path。
- 把 mock、demo fallback、stale cache、paper trading、shadow record 当成真实 live 能力。
- 无报告和可复现证据就口头批准。

### 6.5 Reviewer Agent

Reviewer Agent 必须检查：

```text
是否符合需求
是否符合架构
模块边界是否正确
是否遵守 /product/** API 规则
是否有数据质量字段
fallback 是否受控
是否影响 restricted modules
测试是否充分
是否泄露 secret
LLM 权限边界是否清晰
是否伪造 live 数据
是否绕过真实交易安全边界
```

发现 S0/S1/S2 缺陷必须 request changes。

### 6.6 Acceptance Agent

Acceptance Agent 必须检查：

```text
用户需求是否满足
中文 requirement/design/dev/test/review/acceptance 文档是否齐备
人工合并边界是否保留
是否未经许可 auto-merge
是否暴露 LEVEL_3_AUTO
是否未经批准新增真实 order path
artifact 和报告证据是否完整
```

除非需求明确为 mock-only，否则 Acceptance Agent 不得批准只在 mock 模式下工作的功能。

---

## 7. 标准流水线

本仓库采用文档驱动流水线：

```text
User request
  -> PM requirement document
  -> Architect design document
  -> OpenCode team phase plan when Team Pipeline V2 is active
  -> OpenCode phase implementation + self-test + dev report
  -> OpenCode phase verification + test report
  -> OpenCode team lead review after all phases pass
  -> Architect code review
  -> PM acceptance
  -> log update + merge/release
```

每个 stage gate 都必须产出所需文档后才能推进。如果某阶段失败，必须返回负责的前一阶段，而不是绕过流程补丁式修复。

分支与并行工作必须遵守 `docs/process/BRANCH_WORKFLOW.md`。

简化规则：

```text
main 保持稳定
epic/<date-feature> 是集成分支
developer 在 feat/<feature>/<module> 工作
tester 在临时本地 test/<feature>/<scope>-<tester>-<timestamp> 分支验证
review fix 在 fix/<feature>/<issue> 分支
```

Test Engineer Agent 不得在原开发分支修改业务代码。

---

## 8. 版本路线执行规则

Agent 必须以 `docs/roadmap/MASTER_ROADMAP.md` 作为高层路线图。

### 8.1 V16 平台基础层

```text
V16.1 AgentOps Control Tower Foundation
V16.2 Market Data Relay & Provider Contract
V16.3 Provider Test Suite & Fallback Governance
V16.4 Quant Tool Registry
V16.5 Model Gateway & Research Agent Layer
V16.6 Decision Snapshot & Evidence Engine
V16.7 Position Sizing Engine
V16.8 Strategy Validation Engine & Backtest Tearsheet
V16.9 Risk Sentinel MVP
```

主规则：

```text
数据、工具和证据不可信之前，不得建设交易智能。
```

### 8.2 V16 战略 Alpha 研究层

```text
V16.10 Strategic Alpha Map & Industry Ontology
V16.11 Fundamental Data Relay & Filing Intelligence
V16.12 Alpha Evidence Engine & Company Scoring
V16.13 Industry Chain Tracking & Catalyst Monitor
V16.14 Fundamental Alpha Portfolio & Watchlist
```

主规则：

```text
不输出买卖指令。只建设证据、评分、股票池、降级、中文研究报告。
```

### 8.3 V17 交易执行递进

```text
V17.0 Paper Trading & Human Confirmation
V17.1 Broker Readonly Shadow
V17.2 Small Size Human-Confirmed Trading
V17.3 LEVEL_3_AUTO Evaluation
```

主规则：

```text
先 Paper Trading，再 Broker Readonly Shadow，再小额人工确认真实交易。
LEVEL_3_AUTO 只能在长期证据充分后评估。
```

---

## 9. 常用命令

默认使用 WSL / Linux venv 命令：

```bash
git status --short --branch
git diff --stat
./.venv/bin/python -m ruff check <touched-python-files-and-tests>
./.venv/bin/python -m py_compile <touched-src-python-files>
./.venv/bin/python -m pytest <related-test-files> -q --basetemp=runtime/pytest-tmp-<feature>
git diff --check
```

如果任务明确在 Windows-only workspace 运行，必须在报告里记录，并使用等价 Windows 解释器路径。

触碰共享模型、配置、数据契约、risk、execution、backtest、provider hubs、product routes、Tool Registry、Model Gateway、decision snapshots、position sizing、Alpha scoring 或 UI entrypoints 时，需要运行更广泛回归：

```bash
./.venv/bin/python -m pytest tests -q --tb=short --basetemp=runtime/pytest-tmp-<feature>-full
```

如果 broad checks 因无关历史问题失败，必须报告失败，解释为什么无关，并额外运行 touched-scope 命令。不得声称 full-project success。

---

## 10. 启动入口

| Command                          | Purpose                         |
| -------------------------------- | ------------------------------- |
| `python main.py api`             | FastAPI service                 |
| `python main.py dashboard`       | Streamlit dashboard             |
| `python main.py signal`          | One-shot signal generation      |
| `scripts/start.sh` / `start.bat` | One-click start where supported |

FastAPI app factory：

```text
src/api/app.py:create_app()
```

产品路由位于 `/product` 下。

---

## 11. 架构地图

```text
src/
├── api/                  FastAPI routes and app factory
├── product_app/          Product services, feedback, health gates, config
├── data_gateway/         Market data providers, provider hub, mappers
├── risk_engine/          Risk evaluation and kill-switch behavior
├── execution_engine/     Broker abstractions, execution service, order checks
├── backtest_engine/      Event-driven backtest logic
├── factor_engine/        Factor computation and research support
├── strategy_engine/      Scoring and signal generation
├── stock_pool/           Stock universe and eligibility filters
├── agent_orchestrator/   Monitoring and signal orchestration
├── ui_report/            Streamlit dashboards
├── config/               Settings loaded from environment
└── models/               Pydantic schemas
```

未来模块必须保持边界：

```text
src/product_app/agentops/
src/product_app/market_data/
src/product_app/tools/
src/product_app/model_gateway/
src/product_app/decisions/
src/product_app/position_sizing/
src/product_app/backtests/
src/product_app/risk_sentinel/
src/product_app/fundamental/
src/product_app/alpha/
src/product_app/paper_trading/
src/product_app/broker_shadow/
```

关键边界：

- 产品 live workflow 必须通过 `LiveDataService` 或未来 Market Data Relay 进入市场数据。
- Provider priority、fallback、circuit breaking、quality status 属于 provider hub / provider contracts。
- Strategy 和 signal code 不得直接调用 raw providers。
- LLM 必须调用 approved tools，不得任意读内部文件或 raw providers。
- Backtest 和 signal snapshots 必须携带 data source、version、quality status。
- Alpha scoring 必须携带 evidence ids 和 disconfirm evidence ids。

---

## 12. 受限模块

以下区域变更需要额外审查和负向测试：

| Module                             | Extra requirement                                        |
| ---------------------------------- | -------------------------------------------------------- |
| `src/risk_engine/`                 | Risk veto、kill switch、负向测试                         |
| `src/execution_engine/`            | 人工确认、非交易时段、订单状态、黑名单测试               |
| `src/data_gateway/`                | 单位、时区、延迟、fallback、异常数据、fail-closed 测试   |
| `src/backtest_engine/`             | 佣金、滑点、印花税、涨跌停、停牌验证                     |
| `src/factor_engine/`               | 因子命名、类型、证据、LLM 边界测试                       |
| `src/strategy_engine/`             | 股票池过滤、解释、风险提示测试                           |
| `src/product_app/agentops/`        | 只读保证、脱敏、缺失状态测试                             |
| `src/product_app/market_data/`     | Provider contract、cache/fallback、stale/mock 测试       |
| `src/product_app/tools/`           | Tool permission、输入输出 schema、审计日志测试           |
| `src/product_app/model_gateway/`   | Prompt version、输出校验、fallback、forbidden-field 测试 |
| `src/product_app/decisions/`       | Evidence snapshot、label backfill、replay 测试           |
| `src/product_app/position_sizing/` | Fractional Kelly、risk constraints、no-order 测试        |
| `src/product_app/backtests/`       | Artifacts、metrics、data quality、tearsheet 测试         |
| `src/product_app/risk_sentinel/`   | 盘中风险、fail-closed、stale/mock 测试                   |
| `src/product_app/fundamental/`     | Filing parser、source hash、抽取失败测试                 |
| `src/product_app/alpha/`           | Alpha style、evidence、downgrade、BAN 测试               |
| `src/product_app/paper_trading/`   | 人工确认、模拟成交、无真实订单测试                       |
| `src/product_app/broker_shadow/`   | 只读强制、对账、权限失败测试                             |
| `src/api/`                         | HTTP contract、无效参数、secret leakage、安全边界测试    |
| `src/ui_report/`                   | touched user flow 的浏览器或 Streamlit smoke 测试        |

---

## 13. 测试模式

- 测试通常使用 `pytest`、`unittest.mock.patch`、inline fixtures 和 `setup_method`。
- API 测试使用 `fastapi.testclient.TestClient`，目标为 `src.api.app.app` 或 app factory。
- 测试 product routes 时，通过 route-level `_get_*` helpers mock service singleton。
- 外部 Provider 必须在确定性测试中 mock。
- 真实 Provider smoke test 只能作为 acceptance evidence，且必须由架构文档或测试计划明确要求。
- 始终使用 `--basetemp=runtime/pytest-tmp-<feature>` 隔离临时文件。

相关时必须包含负向测试：

```text
empty data
missing fields
invalid symbol
stale data
mock data
fallback data
provider timeout
provider inconsistency
model output invalid
LLM forbidden buy/sell field
risk veto
stock pool BAN
no human confirmation
API permission denial
UI error state
```

---

## 14. 开发报告要求

每份 `docs/features/<feature-id>/phase-<n>-dev-report.md` 开发报告必须包含：

- Requirement document path。
- Architecture document path。
- Roadmap section reference。
- Changed files。
- Feature-to-code mapping。
- Added or updated tests。
- Exact commands and results。
- Data source and data-quality handling。
- API contract impact。
- UI impact。
- Agent / LLM boundary impact。
- Skipped or not-run items with reasons。
- Remaining risks。
- Whether real trading capability is affected。
- Confirmation that risk、stock-pool filtering、human confirmation、provider contracts、Tool Registry、fail-closed behavior were not bypassed。

---

## 15. 测试报告要求

每份 `docs/features/<feature-id>/phase-<n>-test-report.md` 测试报告必须包含：

- Requirement、architecture、development report paths。
- Roadmap section reference。
- Test environment。
- Test scope and out-of-scope items。
- Requirement coverage matrix。
- Commands and results。
- API/UI/CLI/data-source/provider/tool/model smoke evidence when applicable。
- Data-quality and fail-closed evidence。
- Defect list with severity。
- Feedback bug file paths when generated。
- Remaining risk。
- Final result：`PASS`、`PASS_WITH_NOTES` 或 `REJECTED`。

---

## 16. 缺陷等级

| Severity | Meaning                                               | Blocking            |
| -------- | ----------------------------------------------------- | ------------------- |
| S0       | 真实错误下单风险、风控绕过、secret 泄露、严重数据误用 | Always blocking     |
| S1       | 核心功能不可用、主流程损坏、交易状态错误              | Always blocking     |
| S2       | 重要部分失败、覆盖不足、错误 fallback、fake live data | Blocking by default |
| S3       | 非核心 UX 或低风险文档问题                            | May pass with notes |
| S4       | 建议、重构、性能优化                                  | Non-blocking        |

额外 S0/S1 示例：

```text
LLM 直接写 final buy/sell/order 字段
mock 数据显示为 live
stale 数据未经许可进入 signal path
risk veto 被绕过
stock pool BAN 被忽略
broker readonly path 发出真实订单
secret 出现在日志、报告或 UI
```

---

## 17. 环境

- Python 3.10+
- Virtual environment：`.venv/`
- Configuration：从 `.env.example` 复制 `.env`，由 `python-dotenv` 加载
- Secrets：只允许环境变量
- Ruff config：`pyproject.toml`
- Static checks：除非任务另有说明，否则运行 ruff 和 py_compile

---

## 18. 文档规则

- 根级 `AGENTS.md` 保持通用。
- 功能特定指令放在 `docs/features/<feature-id>/architecture.md` 或相关 feature 目录文档。
- 验收标准放在 `docs/features/<feature-id>/requirements.md` 和 `docs/features/<feature-id>/acceptance.md`。
- Developer evidence 放在 `docs/features/<feature-id>/phase-<n>-dev-report.md`。
- Tester evidence 放在 `docs/features/<feature-id>/phase-<n>-test-report.md`。
- 只有阶段或发布状态实际变化时，才更新 `docs/log/DEVELOPMENT_LOG.md` 和 `docs/log/PHASE_COMPLETION_REPORT.md`。
- 不得把 canonical handoff instructions 放在 scratchpad 目录。
- 除非 `docs/roadmap/MASTER_ROADMAP.md` 已覆盖旧内容或已归档，否则不得删除旧 roadmap 文件。

---

## 19. 中文输出要求

- 用户可见输出和新功能文档默认使用中文。
- 非纯文档 PR 必须在 diff 中包含 `docs/features/<feature-id>/phase-<n>-dev-report.md` 中文功能说明和 `docs/features/<feature-id>/acceptance.md` 中文验收报告。
- 报告必须包含：变更范围、测试命令、测试结果、安全确认、最终结论。
- 代码标识、JSON key、环境变量和第三方术语保留英文。

---

## 20. Reviewer 不可跳过检查清单

任何 PR 批准前，Reviewer / Acceptance Agent 必须回答：

```text
是否符合 docs/roadmap/MASTER_ROADMAP.md？
产品 API 是否仍位于 /product/**？
是否保留 Streamlit，除非 React 基线已明确批准？
是否使用 Provider 契约，而不是 raw data source？
相关响应是否暴露 data quality？
stale/mock/fallback 是否在 signal/trading 路径 fail closed？
LLM 是否只做研究/解释，不越权？
是否避免未经授权新增真实订单能力？
是否包含 normal 和 negative paths 测试？
是否包含中文 dev/test/acceptance 证据？
是否保留 manual merge 和 human confirmation 边界？
```

只要任一答案为否，除非当前 requirement 和 architecture 文档明确解释并批准例外，否则 PR 必须被拒绝或退回修复。

---
> Source: [dafienoly/quant-trading-agent](https://github.com/dafienoly/quant-trading-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
