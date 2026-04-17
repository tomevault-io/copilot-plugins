## momentum-finance

> MomentumFinance is an AI-powered personal finance engine that provides autonomous budget analysis and investment strategy insights.

# MomentumFinance: Agentic Grounding (Feb 2026)

## Purpose

MomentumFinance is an AI-powered personal finance engine that provides autonomous budget analysis and investment strategy insights.

## Core Objectives

1. **Budget Optimization**: Analyze spending patterns and propose proactive savings strategies.
2. **Strategy Generation**: Use `BudgetAgent` to derive actionable financial advice from raw transaction data.

## Agent Instructions

- **Data Privacy**: Never export raw transaction data; only summarize insights.
- **Accuracy**: Prioritize mathematical precision in budget calculations.

## Constraints

- Must use `SharedKit` for agent result standardization.
- Gated approval (`requiresApproval`) is required for any suggested portfolio allocations.

## March 2026 Code Standards

- **No Stubs/Mocks**: Do not use placeholders, mocks, or stubs in implementation or testing.
- **Production Ready**: Every new line of code must be real, working, and production-ready.
- **End-to-End Testing**: Use real working and tested code to verify behavior natively to avoid down-the-line problems.
- **Modern Standards**: Adhere strictly to the latest ecosystem standards (Swift 6.2 concurrency, Python 3.13).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dboone323) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
