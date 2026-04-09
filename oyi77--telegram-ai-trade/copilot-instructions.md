## telegram-ai-trade

> Core trading system rules for AI-powered automated trading bot with MT5 integration


{
  "trading_system": {
    "core_components": {
      "ai_analyzer": {
        "input_sources": ["OHLCV_data", "news_sentiment", "market_session", "liquidity_data"],
        "output_format": "JSON matching app-code-prompt.json signal_schema",
        "confidence_threshold": 60,
        "analysis_timeout_ms": 5000,
        "required_fields": ["symbol", "bias", "setups", "entry_zone", "sl", "tp", "confidence"]
      },
      "signal_processor": {
        "validation_rules": [
          "Schema compliance check",
          "Confidence threshold validation",
          "Risk parameter validation",
          "Market hours validation"
        ],
        "deduplication": {
          "enabled": true,
          "ttl_minutes": 90,
          "key_fields": ["symbol", "bias", "entry_zone"]
        }
      },
      "risk_manager": {
        "position_sizing": "risk_percent_of_equity_based_on_SL_distance",
        "max_risk_per_trade_pct": 2.0,
        "max_daily_drawdown_pct": 6.0,
        "consecutive_loss_rules": [
          {"losses": 2, "action": "reduce_size_50_percent"},
          {"losses": 3, "action": "pause_and_review"}
        ],
        "session_filters": ["avoid_high_impact_news", "prefer_London_NY_overlap"]
      },
      "execution_engine": {
        "platform": "MT5",
        "order_types": ["limit", "market", "stop"],
        "magic_number": 1001,
        "slippage_points": 10,
        "execution_rules": [
          "Never widen stop loss",
          "Move to breakeven at R1",
          "Partial TP at RR 1.5/3.0",
          "Trailing stop enabled"
        ]
      }
    },
    "data_requirements": {
      "timeframes": ["H4", "H1", "M15", "M5", "M1"],
      "data_sources": ["MT4/MT5", "Binance", "Bybit", "News_API"],
      "real_time_requirements": "true",
      "historical_context": "minimum_1000_candles",
      "timezone": "Asia/Jakarta"
    },
    "performance_targets": {
      "signal_generation": "< 500ms",
      "execution_latency": "< 100ms",
      "system_uptime": "> 99.5%",
      "max_memory_usage": "2GB",
      "max_cpu_usage": "70%"
    }
  },
  "mt5_integration": {
    "connection": {
      "retry_attempts": 3,
      "connection_timeout": 30,
      "heartbeat_interval": 60,
      "auto_reconnect": true
    },
    "order_management": {
      "position_tracking": true,
      "real_time_updates": true,
      "order_modification": true,
      "partial_close": true
    },
    "risk_controls": {
      "max_positions": 10,
      "max_daily_trades": 50,
      "emergency_stop": true,
      "circuit_breaker": true
    }
  },
  "ai_analysis_workflow": {
    "input_processing": [
      "Market data normalization",
      "News sentiment scoring",
      "Session timing validation",
      "Liquidity zone identification"
    ],
    "analysis_steps": [
      "H4 big picture analysis",
      "H1 structure confirmation",
      "M15 entry zone refinement",
      "M5 execution timing",
      "M1 trigger confirmation"
    ],
    "output_validation": [
      "Schema compliance",
      "Confidence scoring",
      "Risk parameter check",
      "Execution feasibility"
    ]
  },
  "error_handling": {
    "critical_errors": ["MT5_connection_lost", "signal_validation_failed", "execution_failed"],
    "recovery_actions": {
      "MT5_connection_lost": "auto_reconnect_with_exponential_backoff",
      "signal_validation_failed": "log_and_skip",
      "execution_failed": "retry_with_reduced_size"
    },
    "logging": {
      "level": "INFO",
      "format": "JSON",
      "required_fields": ["timestamp", "level", "component", "message", "trace_id"]
    }
  },
  "monitoring": {
    "metrics": [
      "trades_per_hour",
      "win_rate",
      "profit_factor",
      "max_drawdown",
      "system_latency",
      "error_rate"
    ],
    "alerts": {
      "error_rate_threshold": "1%",
      "latency_threshold": "1000ms",
      "drawdown_threshold": "5%",
      "connection_failure_threshold": "3"
    }
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oyi77)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/oyi77)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
