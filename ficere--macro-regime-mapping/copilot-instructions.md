## macro-regime-mapping

> 宏观推演：当用户需要宏观经济复盘、全球宏观情景推演、政策周期分析、大类资产敏感性、通胀/流动性/地缘事件分析、预期差识别、触发条件、失效信号和风险控制框架时使用。输出为非投资建议的结构化宏观研究。


# 宏观推演

## When to Use This Skill

Use this skill when the user asks to:

- Summarize the current global macro environment or monthly macro changes
- Build a structured macro scenario map for any country, region, or global market
- Analyze inflation, liquidity, fiscal policy, monetary policy, exchange rates, geopolitics, commodities, bonds, equities, or real estate as interacting variables
- Translate macro variables into non-advisory asset-class implications
- Identify expectation gaps, trigger points, invalidation conditions, and follow-up indicators
- Review prior macro calls and update a thesis based on new evidence
- Produce annual or quarterly macro forecasts with clear assumptions, confidence levels, and invalidation criteria
- Diagnose an existing portfolio by overlaying macro regime on position-level quantitative scores, combining industry strength rankings, trend resonance, and risk alerts
- Backtest a quantitative stock-picking pipeline and compare its win rates against macro regime periods

Do not use this skill for personalized financial advice, direct buy/sell instructions, or claims of guaranteed returns.

This skill can be combined with quantitative stock-scoring pipelines (e.g., a-stock-analyst's 7-stage scoring) for portfolio diagnosis — see `references/portfolio-diagnosis.md`.

## Core Principle

Treat macro analysis as a dynamic regime map, not a single forecast. Each output should connect:

1. Current state and the dominant macro axis
2. Key drivers and stakeholder incentives
3. Transmission channels
4. Data anchors and policy signals
5. Scenario branches
6. Asset-class sensitivity
7. Confirmation and invalidation signals
8. Risk controls and uncertainty

## Quick Start / 快速上手

三步触发本 Skill：

### Step 1: 说出你要什么 / Tell the Agent What You Need

使用自然语言说出分析需求，可选指定输出模板：

```text
"做一份 2026 年 4 月全球宏观月度复盘"          → 自动匹配「月度宏观复盘」模板
"分析 PBOC 降准 50bp 对人民币和 A 股的影响"    → 自动匹配「事件冲击分析」模板
"检查我的观点：今年美国通胀会回落到 2%"         → 自动匹配「资产观点检查」模板
"帮我看一下当前持仓是否需要调整"                → 自动匹配「组合诊断」模板 + portfolio-diagnosis.md
"美联储今年会降息吗？"                          → 自动匹配「轻量查询」模板
```

### Step 2: Agent 自动路由 / The Agent Picks the Right Depth

Agent 会根据问题复杂度自动选择两套流程之一：

- **轻量流程**：单一问题、快速判断、暂不需要完整情景表 → 精简版 5 步（主导轴 + 核心传导 + 1 情景 + 置信度 + 风险声明）
- **深度流程**：多资产/多地区/多情景/月度或季度展望 → 完整 12-13 步推演工作流

详见下方「Required Workflow → 1a. 路由判断」。

### Step 3: 对齐输出结构 / Align with Output Templates

Agent 输出将自动对齐到本 Skill 中定义的模板结构。首次使用时，可要求 Agent 先加载 `references/examples.md` 查看完整报告范例来对齐风格。

## Required Workflow

### 1. Define the Time Window and Output Mode

Clarify the observation window:

- Monthly review
- Quarterly outlook
- Event-driven update
- Annual scenario map
- Asset-specific follow-up

If the user does not specify a window, default to the current month and the next one to three quarters.

Then identify the output mode:

- Review mode: explain what changed and why
- Forecast mode: state directional conclusions with assumptions and confidence levels
- Shock mode: analyze event transmission and second-order effects
- Thesis-check mode: test a user's claim against data, policy, positioning, and counterarguments
- Portfolio-diagnosis mode: overlay macro regime on position-level scores → see `references/portfolio-diagnosis.md`

### 1a. Route: Lightweight vs Deep / 流程路由（v1.2 新增）

在进入完整工作流之前，根据查询复杂度分流：

**触发轻量流程的条件（满足 ≥2 条即走轻量）：**
- 单一问题 / 单一资产 / 单一政策事件
- 用户要求"快速判断"、"一句话结论"、"简单说说"
- 仅需方向性结论，不需要完整情景表
- 问题可以在 3 条因果链内回答完毕
- 非月度/季度/年度展望类任务

**触发深度流程的条件（满足任一条即走深度）：**
- 月度/季度/年度宏观复盘或展望
- 多地区、多资产、跨资产类别分析
- 用户要求"详细分析"、"完整报告"、"情景推演"
- 需要输出情景分支表（base/upside/downside/tail risk）
- 组合诊断类任务
- 用户明确要求使用完整工作流

**轻量流程（5 步精简版）：**
1. 确认主导宏观轴
2. 数据与叙事分离 → 锚定核心变量
3. 简化传导链（1 条主链即可）
4. 1 个基准情景 + 置信度 + 失效条件（无需完整 4 情景）
5. 风险声明

输出使用轻量模板（见下方「Output Templates → Lightweight / Quick Query」）。

**深度流程**：继续下方完整 2→13 步工作流。

### 2. Identify the Dominant Macro Axis

Start by asking which structural force is currently driving the system:

- Central-bank rate cycle: tightening, pause, easing, or re-acceleration
- Dollar liquidity and credit cycle: expansion, contraction, refinancing stress, or reserve reallocation
- Fiscal cycle: stimulus, austerity, front-loading, debt-service pressure, or deficit constraint
- Geopolitical or trade-order shift: globalization, fragmentation, tariffs, sanctions, supply-chain relocation
- Debt and balance-sheet cycle: leverage expansion, deleveraging, refinancing wall, collateral stress
- Technology or industrial narrative: AI capex, energy transition, industrial upgrading, productivity shock

Treat local variables as downstream functions of the dominant axis unless there is strong evidence of local decoupling. Explicitly state what would mark a regime switch.

### 3. Separate Hard Data from Narrative

Classify inputs into:

- Hard data: inflation, PMI, GDP, employment, fiscal issuance, credit, property sales, trade, inventories, interest rates, exchange rates
- Policy signals: central bank language, fiscal stance, regulatory intervention, industrial policy, election incentives
- Market behavior: price action, factor rotation, flows, leverage, positioning, sentiment
- Narrative: consensus stories, media framing, market slogans, geopolitical rhetoric

Give higher weight to data, policy constraints, and cross-asset confirmation than to narratives.

### 4. Build the Domestic or Regional Macro Block

For domestic or region-specific analysis, cover the relevant local variables:

- Fiscal impulse: government bond issuance pace, front-loading, local-government capacity, debt-service pressure
- Monetary and credit conditions: rate-cut probability, liquidity, social financing, deposit rates, bank behavior
- Inflation and industrial prices: CPI, PPI, upstream/downstream pass-through, commodity cost pressure
- Real estate: transaction volume, price trend, city-tier divergence, policy stimulus, supply-demand balance
- Three growth engines: export, investment, consumption
- Corporate profit structure: which sectors drive earnings improvement, whether profit recovery is broad or narrow

Always ask whether price recovery is demand-driven, cost-driven, or liquidity-driven.

**For China-specific analysis**, load `references/china-macro.md` to access the China macro variable library — covering MLF/OMO/PSL/再贷款, TSF/M2/M1/LPR, 城投债/隐性债务, 土地出让金, 房地产调控, 出口/汇率管理, and 产业政策 — each with incentive-constraint-transmission mapping isomorphic to the stakeholder analysis framework in Step 6.

### 5. Build the External Macro Block

For global analysis, cover:

- US cycle: inflation, employment quality, household balance sheet, fiscal impulse, Fed reaction function, political calendar
- Japan cycle: fiscal expansion, bond yields, yen pressure, equity-market policy support, inflation persistence
- Other regions: energy sensitivity, trade relations, industrial competitiveness, political risk
- Geopolitics: conflict probability, escalation path, negotiation incentives, supply-chain impact
- Cross-border flows: dollar strength, reserve allocation, capital flight or repatriation, trade settlement behavior

Treat political events as economic incentives with deadlines, stakeholders, and negotiation payoffs.

### 6. Analyze Stakeholder Incentives

For each major policy or event question, list the relevant stakeholders and constraints:

- Government or fiscal authority: growth target, employment, debt service, social stability, election or term horizon
- Central bank: inflation target, currency stability, financial stability, credibility, political pressure
- Enterprises: margins, inventory, financing cost, export orders, capex incentives
- Households: income expectation, debt burden, wealth effect, consumption willingness
- Foreign capital or external counterparties: yield spread, geopolitical risk, hedging cost, exit constraints

Do not predict personal psychology. Infer likely behavior from incentives, constraints, and available choices.

### 7. Map Transmission Channels

Use explicit causal chains. Examples:

- Export strength → trade surplus → settlement demand → currency appreciation pressure → exporter margin pressure
- Government-bond front-loading → early-year investment support → mid-year fiscal fade → growth slowdown risk
- Energy shock → input costs → CPI/PPI split → central-bank reaction → bond yield repricing
- Geopolitical shock → risk-off flows → safe-haven demand → liquidity tightening → leveraged-asset drawdown
- Upstream commodity rally → downstream margin compression → production slowdown → rally sustainability test

Avoid isolated conclusions that do not show the mechanism.

### 8. Anchor with Data and Pricing

Use data to locate the current stage and remaining space, not to create false point precision:

- Stock versus flow: debt/GDP, housing inventory, credit stock, fiscal balance
- Rate of change: acceleration, deceleration, inflection, base effect
- Historical percentile: valuation, yield, spread, volatility, inventory
- Policy rate versus market-implied rate: whether easing/tightening is already over-priced
- Supply-demand gap: new supply, inventory digestion, replacement demand, subsidy pull-forward

When market pricing has fully discounted a story, treat the story as vulnerable even if the direction is logically correct.

### 9. Decode Policy Signals

Read policy through actions, omissions, and implementation details:

- Additive signal: a new policy reveals the problem being prioritized
- Subtractive signal: a stopped or missing policy reveals what is being abandoned or deprioritized
- Action versus rhetoric: when words conflict with implementation, prioritize actions
- Preparatory signal: small technical measures may be laying the groundwork for larger regime changes

Do not over-read slogans. Tie every policy interpretation to budget constraints, enforcement capacity, and observed follow-through.

### 10. Create Scenario Branches

For each major uncertainty, produce 2-4 branches:

- Base case
- Upside surprise
- Downside surprise
- Tail risk, if relevant

For every branch, state:

- Probability as qualitative or broad percentage bands if the user requests probabilities
- Trigger conditions
- Expected macro result
- Asset-class implications
- Invalidation signal

Never present one scenario as inevitable.

### 11. Analyze Asset-Class Sensitivity

Translate macro conclusions into asset-class sensitivity, not direct investment advice:

- FX: interest-rate differentials, trade surplus, settlement behavior, safe-haven flow, policy tolerance
- Bonds: growth slowdown, inflation pressure, supply-demand balance, central-bank reaction function
- Equities: earnings breadth, liquidity, policy support, valuation, leverage, factor rotation
- Commodities: physical supply-demand, financial speculation, inventory, geopolitical premium, pass-through ability
- Real estate: policy easing, mortgage rates, supply, household income expectations, city-tier divergence
- Defensive yield assets: dividend durability, bond-like valuation, rate alternatives, crowding risk

Use the language of "sensitivity," "watch points," and "risk-reward," not "must buy" or "must sell."

### 12. Identify Expectation Gaps

Look for gaps between market consensus and observable constraints:

- Consensus expects policy easing, but policy language suggests restraint
- Market prices conflict escalation, but actors have incentives to negotiate
- Commodity prices imply demand boom, but downstream orders and inventories do not confirm it
- Equity rally assumes broad earnings recovery, but profits are concentrated in a few sectors
- Safe-haven asset fails to rise during stress, indicating that its regime may have changed

Expectation gaps should include both the opportunity and the reason they may be wrong.

### 13. Add Forecast Discipline and Risk Controls

Every output with asset implications must include:

- What would prove the thesis wrong
- Which data should be monitored next
- Which part is high confidence and which part is speculative
- Direction confidence and timing confidence separately when making forecasts
- Key assumptions and their fragility
- Whether the asset is driven by fundamentals, liquidity, policy, or crowd behavior
- A reminder that this is not personalized financial advice

For leveraged, futures, options, or inverse products, explicitly warn that they can suffer rapid losses and are unsuitable for most users.

When the user asks for explicit predictions, use this format:

```text
【判断】一句话给出方向性结论
【核心逻辑】1-3 条因果链
【关键前提】该判断成立所需的必要条件
【失效条件】什么证据会证伪该判断
【置信度】方向置信度：高/中/低；时间节点置信度：高/中/低
```

## Output Templates

### Lightweight / Quick Query（v1.2 新增）

Use this structure when the query matches the lightweight routing criteria (see Step 1a):

```text
## 【快速判断】<一句话方向性结论>

### 当前主导轴
<1-2 句识别当前驱动该问题的最主要宏观力量>

### 核心传导链
<1 条关键因果链，不超过 3 步>

### 基准情景
<最可能路径 + 触发条件 + 资产敏感性概述>

### 置信度与跟踪
【方向置信度】高/中/低
【失效条件】<什么证据会推翻此判断>
【跟踪信号】<1-2 个可观测的后续指标>

### ⚠️ 声明：本分析为宏观研究方法论框架，不构成投资建议。
```

### Monthly Macro Review

Use this structure:

1. Executive summary: three to five key changes
2. Domestic block: fiscal, monetary, inflation, property, trade, consumption/investment
3. Global block: US, Japan, Europe, geopolitics, dollar/liquidity
4. Asset sensitivity map: FX, bonds, equities, commodities, real estate/yield assets
5. Scenario table: base/upside/downside/tail risk
6. Watchlist: next data releases, policy meetings, events, invalidation signals
7. Risk disclaimer

### Event Shock Analysis

Use this structure:

1. Event summary and immediate market reaction
2. Actual economic transmission versus emotional pricing
3. Stakeholder incentives and negotiation path
4. Scenario branches with triggers
5. Asset sensitivity and likely second-order effects
6. What would change the conclusion

### Asset Thesis Check

Use this structure:

1. Asset and current consensus
2. Macro drivers supporting the thesis
3. Macro drivers against the thesis
4. Cross-asset confirmation or contradiction
5. Trigger/invalidation checklist
6. Risk-reward assessment without direct advice

### Portfolio Diagnosis（v1.2 新增）

Use this structure when the user asks about a set of existing holdings and wants operation recommendations (combines macro context with position-level quantitative scores):

1. Current holdings table with quant scores, signals, trends, and risk levels
2. Macro context: market breadth, industry strength ranking, market style
3. Cross-reference matrix: map each position against macro context to find misalignments
4. Per-position operation plan (减仓/清仓/保留/加仓) with reasoning and stop-loss
5. Reallocation map: show where freed-up capital should go
6. Scenario branching: portfolio outcome under base/upside/downside/tail risk
7. Invalidation triggers: conditions that would pause or reverse the recommendation

See `references/portfolio-diagnosis.md` for the full methodology with SQL queries and templates.

## Reference Loading

Read `references/methodology.md` when:

- The user asks for a full framework, reusable methodology, or detailed report
- The task involves multiple countries or multiple asset classes
- The output needs a scenario table, watchlist, or investment-style review

Read `references/reasoning-framework.md` when:

- The user asks for annual forecasts, quarterly predictions, or explicit directional calls
- The output needs stakeholder incentives, data anchoring, policy-signal decoding, or confidence grading
- The task involves distinguishing trend direction from timing or identifying common reasoning biases

Read `references/china-macro.md` (v1.2 新增) when:

- The user asks about China-specific monetary tools (MLF/OMO/PSL/LPR), credit aggregates (TSF/M2/M1), local government debt, land sales, real estate policy, or industrial policy
- The task involves China domestic macro analysis that requires understanding of the three parallel policy systems (price-based / quantity-based / administrative)
- The output needs incentive-constraint-transmission mapping for Chinese policy variables
- The user asks about China exchange-rate management tools (逆周期因子/离岸央票/窗口指导)

Read `references/examples.md` (v1.2 新增) when:

- The user asks to see a full example of how a macro output should look
- The task involves monthly macro review or single policy event analysis and the user wants to align on output structure first
- The user is a new user wanting to understand the expected report format and quality
- The agent needs a reference for how to organize and format the final output

Read `references/portfolio-diagnosis.md` (v1.2 新增) when:

- The user asks for portfolio operation recommendations based on existing holdings
- The task combines macro regime overlay with quantitative stock scores
- The user wants cross-reference between macro context and position-level signals
- Backtest validation of stock-picking pipeline performance across macro regimes is needed

## Style Guide

- Respond in the user's language when possible, and support bilingual Chinese-English outputs when useful.
- Be structured, concise, and explicit about causal chains.
- Use tables when comparing scenarios, variables, or assets.
- Do not imitate any creator's personal voice, catchphrases, paywall structure, or identity.
- Avoid sensational language. Replace certainty with conditional reasoning.
- Cite current external data sources when using web or finance tools.

## Safety Boundary

This skill provides macro research and educational analysis only. It does not provide personalized investment advice, portfolio management, tax advice, or trading instructions.

---
> Source: [Ficere/macro-regime-mapping](https://github.com/Ficere/macro-regime-mapping) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
