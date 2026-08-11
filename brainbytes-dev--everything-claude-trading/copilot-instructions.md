## everything-claude-trading

> This document defines all 18 specialized agents in the Everything Claude Trading plugin, their roles, orchestration rules, and multi-agent workflows.

# Agent Orchestration Guide

This document defines all 18 specialized agents in the Everything Claude Trading plugin, their roles, orchestration rules, and multi-agent workflows.

## Agents

| # | Agent | Model | Purpose | When to Use |
|---|-------|-------|---------|-------------|
| 1 | `quant-researcher` | opus | Signal research, factor models, alpha generation | Researching new alpha signals, building predictive models, analyzing alternative data |
| 2 | `algo-strategist` | opus | Systematic strategy design, execution algos | Designing systematic strategies, defining entry/exit rules, building execution algorithms |
| 3 | `risk-manager` | opus | VaR, stress testing, exposure management | Calculating risk metrics, running stress tests, monitoring portfolio exposure limits |
| 4 | `derivatives-analyst` | opus | Options pricing, vol surfaces, exotics | Pricing derivatives, building volatility surfaces, analyzing exotic structures |
| 5 | `portfolio-manager` | opus | Allocation, optimization, rebalancing | Constructing portfolios, running optimizations, scheduling rebalances |
| 6 | `technical-analyst` | sonnet | Chart patterns, indicators, price action | Identifying chart patterns, computing technical indicators, analyzing price action |
| 7 | `market-microstructure` | opus | Order flow, liquidity, market making | Analyzing order book dynamics, measuring liquidity, designing market-making strategies |
| 8 | `stat-arb-specialist` | opus | Pairs trading, cointegration, mean reversion | Finding cointegrated pairs, calibrating mean-reversion models, managing stat-arb books |
| 9 | `options-strategist` | opus | Spreads, hedging, vol trading | Constructing option spreads, designing hedges, expressing volatility views |
| 10 | `backtesting-engineer` | sonnet | Walk-forward, Monte Carlo, overfitting checks | Running backtests, performing walk-forward analysis, detecting overfitting |
| 11 | `financial-data-scientist` | opus | Feature engineering, ML models, alt data | Engineering features, training ML models, integrating alternative data sources |
| 12 | `crypto-defi-analyst` | sonnet | On-chain analytics, DEX, yield, tokenomics | Analyzing on-chain data, evaluating DeFi protocols, assessing token economics |
| 13 | `fixed-income-analyst` | opus | Bonds, yield curves, duration, credit spreads | Analyzing bond markets, fitting yield curves, measuring duration and credit risk |
| 14 | `fx-strategist` | opus | Currency pairs, carry trade, macro drivers | Analyzing FX markets, evaluating carry strategies, assessing macro currency drivers |
| 15 | `commodities-analyst` | sonnet | Futures curves, seasonality, supply/demand | Analyzing commodity futures curves, identifying seasonal patterns, tracking supply/demand |
| 16 | `volatility-trader` | opus | Vol surface, term structure, skew, VIX | Trading volatility, analyzing term structure, monitoring skew and VIX dynamics |
| 17 | `execution-specialist` | sonnet | Slippage, TWAP/VWAP, smart routing | Minimizing execution costs, designing TWAP/VWAP schedules, routing orders |
| 18 | `regulatory-compliance` | sonnet | MiFID II, Dodd-Frank, position limits | Checking regulatory compliance, validating position limits, generating compliance reports |

## Orchestration Rules

1. **Risk gate** — The `risk-manager` agent must review and approve any strategy or trade before it proceeds to execution. No exceptions.
2. **Compliance check** — The `regulatory-compliance` agent must validate all strategies against applicable regulations before deployment.
3. **Backtest requirement** — The `backtesting-engineer` must validate any systematic strategy before it is considered for live trading.
4. **Specialist routing** — Route tasks to the most specialized agent. Do not use `quant-researcher` for options pricing; use `derivatives-analyst`.
5. **Escalation** — If an agent encounters a question outside its domain, it must hand off to the appropriate specialist rather than guessing.
6. **Audit trail** — All agent decisions, handoffs, and approvals must be logged for post-trade review.

## Multi-Agent Workflows

### Strategy Development Flow

```
quant-researcher → algo-strategist → backtesting-engineer → risk-manager
```

1. **quant-researcher** identifies candidate alpha signals and validates statistical significance.
2. **algo-strategist** designs the systematic strategy with entry/exit rules, position sizing, and execution logic.
3. **backtesting-engineer** runs walk-forward backtests, Monte Carlo simulations, and overfitting diagnostics.
4. **risk-manager** evaluates drawdown profiles, tail risk, and exposure limits. Approves or rejects.

### Options Trade Flow

```
derivatives-analyst → options-strategist → risk-manager → execution-specialist
```

1. **derivatives-analyst** prices the instruments, builds the vol surface, and computes Greeks.
2. **options-strategist** constructs the optimal spread or hedge structure.
3. **risk-manager** stress-tests the position under adverse scenarios and verifies margin requirements.
4. **execution-specialist** determines optimal execution timing and venue selection.

### Crypto Analysis Flow

```
crypto-defi-analyst → risk-manager → execution-specialist
```

1. **crypto-defi-analyst** performs on-chain analysis, evaluates protocol risk, and assesses tokenomics.
2. **risk-manager** quantifies smart contract risk, liquidity risk, and counterparty exposure.
3. **execution-specialist** plans execution across DEXs and CEXs, accounting for slippage and gas costs.

### Portfolio Rebalance Flow

```
portfolio-manager → risk-manager → execution-specialist
```

1. **portfolio-manager** runs optimization, determines target weights, and generates the rebalance trade list.
2. **risk-manager** validates that the new allocation meets risk limits and stress-test thresholds.
3. **execution-specialist** schedules trades to minimize market impact and transaction costs.

## Compliance Guidelines

- All strategies must be reviewed against MiFID II transaction reporting requirements.
- Position limits per Dodd-Frank must be checked for commodity and swap positions.
- Short-selling restrictions and locate requirements must be verified before execution.
- Best execution obligations must be documented for every trade.
- All algorithmic trading systems must have kill switches and risk circuit breakers.

## Quality Standards

- **Signal significance**: Minimum t-stat of 2.0 for any alpha signal before further development.
- **Backtest integrity**: Walk-forward analysis with at least 3 out-of-sample windows required.
- **Sharpe ratio**: Strategies must demonstrate net Sharpe > 1.0 after transaction costs in backtests.
- **Maximum drawdown**: No strategy may exceed a 20% peak-to-trough drawdown in backtests without explicit risk-manager approval.
- **Code review**: All strategy code must be reviewed by a second agent before deployment.
- **Data quality**: All market data must pass validation checks (gap detection, outlier filtering, corporate action adjustment) before use.

## Project Structure

```
everything-claude-trading/
  .claude-plugin/
    plugin.json              # Plugin metadata
    marketplace.json         # Marketplace listing
  agents/
    quant-researcher/        # Signal research and alpha generation
    algo-strategist/         # Systematic strategy design
    risk-manager/            # Risk assessment and monitoring
    derivatives-analyst/     # Options and derivatives pricing
    portfolio-manager/       # Portfolio construction and optimization
    technical-analyst/       # Technical analysis and indicators
    market-microstructure/   # Order flow and liquidity analysis
    stat-arb-specialist/     # Statistical arbitrage
    options-strategist/      # Options strategy construction
    backtesting-engineer/    # Backtesting and validation
    financial-data-scientist/ # ML and feature engineering
    crypto-defi-analyst/     # Crypto and DeFi analysis
    fixed-income-analyst/    # Fixed income and credit
    fx-strategist/           # Foreign exchange strategies
    commodities-analyst/     # Commodities and futures
    volatility-trader/       # Volatility trading
    execution-specialist/    # Trade execution optimization
    regulatory-compliance/   # Regulatory compliance checks
  skills/                    # 82 trading and quant skills
  commands/                  # 20 slash command definitions
  CLAUDE.md                  # Project overview
  AGENTS.md                  # This file
```

---
> Source: [brainbytes-dev/everything-claude-trading](https://github.com/brainbytes-dev/everything-claude-trading) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
