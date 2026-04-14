## findash

> Findash/Octopus 11-agent architecture; apply when editing agent modules or orchestrator.


# Findash Agent Modules (Octopus Trading Platform)

You are working in a codebase that implements **11 orchestrated AI agents** (M1–M11). Respect each agent’s responsibility and code locations so changes stay consistent and pipeline-safe.

## Agent → Code Map

| Agent ID | Name | Code Paths | Capabilities |
|----------|------|------------|--------------|
| **M1** | Data Collector | `src/data_processing/` (scraping/, ingestion/, collection_tasks), `src/infrastructure/market_data_*.py`, `src/services/market_data_service.py` | web_scraping, api_fetching, market_data, validation |
| **M2** | Data Warehouse | `src/database/`, `src/analytics/warehouse.py` | data_storage, retrieval, validation, indexing |
| **M3** | Real-time Processor | `src/realtime/`, `src/api/endpoints/realtime.py`, `src/api/endpoints/websocket*.py`, `src/infrastructure/market_data_consumer_service.py` | stream_processing, real_time_analysis, alerts, WebSocket |
| **M4** | Strategy Agent | `src/strategies/` (strategy_agent, signal_fusion) | strategy_execution, signal_generation, backtesting |
| **M5** | ML Models | `src/training/`, `src/api/routes/ml_models.py` | prediction, classification, deep_learning |
| **M6** | Risk Manager | `src/risk/`, `src/compliance/` | risk_assessment, portfolio_optimization, compliance |
| **M7** | Price Predictor | `src/prediction/` (advanced_prediction_agent, prophet_service) | time_series_forecasting, prophet, neural_networks |
| **M8** | Paper Trader | `src/trading/` (simulated execution) | simulated_trading, execution_simulation, performance_tracking |
| **M9** | Sentiment Analyzer | `src/analytics/sentiment_agent.py`, `src/alternative_data/` | sentiment_analysis, news_analysis, social_media_monitoring |
| **M10** | Backtester | `src/backtesting/` | historical_testing, performance_analysis, strategy_validation |
| **M11** | Visualizer / Reports | `src/generative/`, frontend charts/dashboards, reporting APIs | chart_generation, dashboard_updates, reporting |

## Orchestrator

- **File**: `src/core/intelligence_orchestrator.py`
- **Class**: `IntelligenceOrchestrator`
- **Concepts**: `submit_task(agent_name, task_type, data, priority)`, `coordinate_pipeline(symbol, analysis_type)`, `get_task_result(task_id)`
- When adding or changing an agent capability, update the `agents` registry in `_initialize_agents()` and keep task_type contracts consistent so pipelines (e.g. M1 → M3 → M9 → M4) keep working.

## Conventions When Editing Agent Code

1. **Logging**: Use the module logger; include `agent_name` or `task_id` in log messages when handling orchestrator tasks.
2. **Errors**: Use project exceptions from `src/core/exceptions.py` (e.g. `TradingError`, `MLModelError`) where applicable.
3. **Async**: Orchestrator and pipelines are async; prefer `async def` and `await` for I/O in agent code that is invoked from the orchestrator.
4. **Contracts**: Task `data` and result payloads should match what the orchestrator and other agents expect; avoid breaking the pipeline (e.g. M1 output consumed by M3/M9).
5. **Config**: Prefer env/config for API keys, URLs, and feature flags; do not hardcode secrets in agent modules.

Refer to `wiki-content/AI-Agents.md` and `docs/archive/BACKEND_ARCHITECTURE.md` for full agent descriptions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/massoudsh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
