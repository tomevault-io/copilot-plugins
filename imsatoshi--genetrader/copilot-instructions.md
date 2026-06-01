## genetrader

> This document describes how Claude Code can interact with the GeneTrader adaptive optimization system.

# Claude Code Integration for GeneTrader

This document describes how Claude Code can interact with the GeneTrader adaptive optimization system.

## Quick Start

```bash
# Check current strategy performance
python run_adaptive.py --strategy GeneTrader --check-only

# Start adaptive monitoring with Agent API
python run_adaptive.py --strategy GeneTrader --api-port 8090
```

## Architecture Overview

GeneTrader uses an on-the-fly optimization system that:
1. **Monitors** live trading performance from Freqtrade
2. **Detects** strategy degradation using statistical methods (CUSUM, SPC)
3. **Optimizes** with time-weighted recent data when degradation is detected
4. **Deploys** safely with shadow trading and gradual rollout
5. **Rollbacks** automatically if new strategy underperforms

## Claude Code Workflow

When the user asks about trading strategy performance or optimization:

### 1. Check Strategy Performance
```bash
python run_adaptive.py --strategy <STRATEGY_NAME> --check-only
```

This returns JSON with:
- `status`: "healthy" or "degraded"
- `degradation_score`: 0-1 (higher = worse)
- `alerts`: List of detected issues
- `recommendation`: Suggested action
- `current_metrics`: Latest performance data

### 2. Analyze Degradation
If degradation is detected, analyze the alerts:
- `PROFIT_DECLINE`: Profit fell below baseline
- `WIN_RATE_DROP`: Win rate dropped significantly
- `DRAWDOWN_INCREASE`: Drawdown exceeds threshold
- `CONSECUTIVE_LOSSES`: Too many losing trades in a row

### 3. Trigger Optimization (if appropriate)
```bash
python run_adaptive.py --strategy <STRATEGY_NAME> --force-optimize
```

Only trigger optimization if:
- Degradation score > 0.3
- At least 20 trades in evaluation window
- More than 3 days since last optimization

### 4. Monitor Optimization Progress
If Agent API is running (port 8090), use:
```bash
curl -H "X-API-Key: <API_KEY>" http://localhost:8090/api/v1/optimization/status
```

### 5. Approve/Reject Deployment
When new strategy is ready for deployment:
```bash
# Approve
curl -X POST -H "X-API-Key: <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"request_id": "<REQUEST_ID>"}' \
  http://localhost:8090/api/v1/deployment/approve

# Reject
curl -X POST -H "X-API-Key: <API_KEY>" \
  -H "Content-Type: application/json" \
  -d '{"request_id": "<REQUEST_ID>", "reason": "Insufficient improvement"}' \
  http://localhost:8090/api/v1/deployment/reject
```

## Decision Guidelines for Claude

### When to Recommend Optimization
- Profit declined > 50% from baseline
- Win rate dropped > 15% from baseline
- Drawdown exceeds 20%
- 5+ consecutive losing trades

### When to Approve Deployment
- New strategy shows > 20% improvement in backtest
- Drawdown is lower or similar
- Win rate is maintained or improved
- Shadow trading validation passed

### When to Reject Deployment
- Improvement < 10%
- Drawdown increased significantly
- Win rate dropped
- Insufficient backtest trades

### When to Recommend Rollback
- Live drawdown exceeds 15%
- 5+ consecutive losses in live trading
- Performance significantly worse than shadow testing

## API Endpoints Reference

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/health` | GET | Health check |
| `/api/v1/status` | GET | System status |
| `/api/v1/metrics?strategy=X&hours=168` | GET | Performance metrics |
| `/api/v1/versions?strategy=X` | GET | Strategy versions |
| `/api/v1/optimization/status` | GET | Optimization status |
| `/api/v1/approvals/pending` | GET | Pending approvals |
| `/api/v1/optimization/trigger` | POST | Trigger optimization |
| `/api/v1/deployment/approve` | POST | Approve deployment |
| `/api/v1/deployment/reject` | POST | Reject deployment |
| `/api/v1/rollback` | POST | Trigger rollback |

## Configuration

Key settings in `ga.json`:
```json
{
  "adaptive_optimization_enabled": true,
  "performance_check_interval_minutes": 5,
  "degradation_check_interval_minutes": 60,
  "reoptimization_trigger_threshold": 0.3,
  "minimum_trades_for_evaluation": 20,
  "minimum_days_between_optimizations": 3,
  "recent_data_weight": 0.7,
  "shadow_trading_hours": 24,
  "auto_rollback_enabled": true,
  "rollback_drawdown_threshold": 0.15,
  "agent_api_enabled": true,
  "agent_api_port": 8090,
  "agent_approval_required_for_deployment": true
}
```

## Files Structure

```
monitoring/           # Live performance monitoring
  freqtrade_client.py   # Freqtrade API client
  performance_monitor.py # Metrics collection
  degradation_detector.py # Statistical detection

deployment/           # Safe deployment
  version_control.py    # Strategy versioning
  strategy_deployer.py  # Deployment pipeline
  rollback_manager.py   # Auto-rollback

adaptive/             # Adaptive optimization
  adaptive_optimizer.py # Main orchestrator
  weighted_optimizer.py # Time-weighted optimization
  scheduler.py          # Rate limiting & scheduling

agent_api/            # External API
  api_server.py         # REST API
  websocket_manager.py  # Real-time updates
  auth.py               # Authentication
```

## Example Session

User: "检查我的策略表现如何"

Claude should run:
```bash
python run_adaptive.py --strategy GeneTrader --check-only
```

Then analyze the output and provide recommendations based on the decision guidelines above.

---
> Source: [imsatoshi/GeneTrader](https://github.com/imsatoshi/GeneTrader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
