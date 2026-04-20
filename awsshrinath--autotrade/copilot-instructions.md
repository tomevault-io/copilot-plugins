## autotrade

> description: Full Product Requirements Document for the Autotrade System

---

description: Full Product Requirements Document for the Autotrade System
globs:
alwaysApply: false
------------------

# Product Requirements Document (PRD): Autotrade System

## 1. Introduction/Overview

The **Autotrade System** is an advanced, AI-enhanced automated trading system designed for personal portfolio growth. It leverages pre-market analysis, real-time market data, dynamic strategy selection, and GPT-powered self-learning to generate consistent, high-precision trades. Its ultimate goal is to maximize profits, ensure continuous system evolution, and adapt to market dynamics autonomously.

## 2. Goals

* Achieve a minimum of 7–10% profit over a month.
* Reduce failed trades by adaptive learning and decision-making.
* Implement a fully automated system with zero manual intervention.
* Integrate RAG and Module Context Protocol (MCP) for contextual self-learning.
* Achieve robust capital preservation during volatile market conditions.
* Deploy self-healing, auto-updating system components via CI/CD and GKE Autopilot.

## 3. User Stories

* **As a trader**, I want the system to dynamically adjust risk based on market conditions, so that capital is preserved during volatile periods.
* **As a developer**, I want the system to auto-recover and resume trades after a crash.

## 4. Functional Requirements

1. **Pre-Market Analysis Module**: Analyze SGX Nifty, Dow Futures, and India VIX to determine market sentiment.
2. **Strategy Selector**: Dynamically choose the best-performing strategy (e.g., VWAP, ORB, Scalp) based on pre-market and real-time data.
3. **Trade Manager**: Execute and monitor trades using Zerodha Kite; log trade data to Firestore and GCS.
4. **Self-Learning via RAG and MCP**: Continuously analyze logs, generate improvement suggestions, and integrate learning into strategies.
5. **GPT-Powered Reflection**: Perform daily end-of-session analysis to suggest improvements in trade logic and strategy performance.
6. **Risk Governor**: Enforce stop-loss levels, maximum trades per day, and trading cut-off times.
7. **Capital Allocation Module**: Dynamically split capital based on total balance and market conditions, with support for aggressive mode.
8. **Options Strike Picker**: Select optimal strikes for NIFTY and BANKNIFTY based on premium, trend, and expiry.
9. **Deployment Infrastructure**: Utilize GKE Autopilot with self-healing pods and Secret Manager for credentials.
10. **CI/CD Pipeline**: Automate code deployment and updates via GitHub Actions.
11. **Dashboard UI**: Provide categorized views of system status, trade logs, and performance metrics.
12. **Error Handling**: Gracefully handle Zerodha token expirations, pod crashes, trade failures, and insufficient funds.

## 5. Non-Goals (Out of Scope)

* Long-term investment strategies.
* Cryptocurrency trading.
* Forex trading.
* Manual trading interfaces.

## 6. Design Considerations

* All features will be integrated into a clean, categorized **Trade Dashboard**.
* Use modern UI components for performance metrics and logs.
* Incorporate error notifications and recovery status indicators.

## 7. Technical Considerations

* **Infrastructure**: GKE Autopilot for scalable, self-healing pods.
* **Secrets Management**: GCP Secret Manager for credentials.
* **Logging**: Firestore and GCS for trade and system logs.
* **Data Source**: NSE real-time data via Zerodha Kite.
* **CI/CD**: GitHub Actions for seamless auto-deployment.
* **Self-Learning**: RAG and Module Context Protocol for continuous strategy refinement.

## 8. Success Metrics

* Achieve a minimum of 7–10% profit monthly.
* Reduce failed trades by at least 30% compared to manual trading.
* Fully autonomous recovery from crashes and system interruptions.
* Daily GPT-based self-improvement logs integrated into trade logic.

## 9. Open Questions

* Should aggressive mode apply a fixed % capital or dynamically adjust based on trade success?
* What is the threshold for pausing trades during extreme volatility (e.g., global crashes)?
* Should the dashboard include predictive alerts (e.g., potential token expiration or margin shortage)?
* When should we consider introducing additional strategies (beyond VWAP, ORB, Scalp)?

---

📁 **Filename:** `.cursor/rules/prd-autotrade.md`
📂 **Format:** Markdown (`.md`)
👥 **Audience:** Junior developers and system architects for the Autotrade project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/awsshrinath) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
