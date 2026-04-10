## telegram-ai-trade

> Comprehensive rules for maintaining project architecture, including detailed guidelines for modularity, extensibility, and documentation.

{
  "architecture_principles": {
    "modularity": {
      "rules": [
        "Each module must have a single responsibility",
        "Modules must communicate through defined interfaces only",
        "No direct database access from business logic",
        "Use dependency injection for components"
      ],
      "folder_structure": {
        "src/": {
          "analysis/": "Market analysis and signal generation",
          "data/": "Data collection and processing",
          "execution/": "Trade execution and management",
          "risk/": "Risk management and monitoring",
          "common/": "Shared utilities and interfaces"
        },
        "tests/": {
          "unit/": "Unit tests for each module",
          "integration/": "Integration tests",
          "e2e/": "End-to-end tests"
        },
        "config/": {
          "dev/": "Development configuration",
          "prod/": "Production configuration",
          "test/": "Test configuration"
        }
      }
    },
    "interfaces": {
      "required_interfaces": [
        "IDataCollector",
        "IAnalyzer",
        "ITradeExecutor",
        "IRiskManager",
        "IMessageBroker"
      ],
      "interface_rules": [
        "Must define clear input/output contracts",
        "Must include error handling",
        "Must be versioned",
        "Must have documentation"
      ]
    },
    "communication": {
      "patterns": [
        "Event-driven for asynchronous operations",
        "Request-response for synchronous operations",
        "Publisher-subscriber for notifications"
      ],
      "message_format": {
        "type": "JSON",
        "required_fields": ["timestamp", "version", "source", "payload"],
        "validation": "JSON Schema"
      }
    }
  },
  "extensibility": {
    "plugin_system": {
      "areas": [
        "Data sources",
        "Analysis strategies",
        "Execution adapters",
        "Risk rules"
      ],
      "requirements": [
        "Must implement base interface",
        "Must include configuration schema",
        "Must provide documentation",
        "Must include tests"
      ]
    },
    "configuration": {
      "format": "YAML",
      "validation": "JSON Schema",
      "environment_specific": true,
      "hot_reload_support": true
    }
  },
  "security": {
    "authentication": {
      "required": true,
      "methods": ["API key", "OAuth2"],
      "token_validation": true
    },
    "authorization": {
      "role_based": true,
      "action_based": true,
      "audit_logging": true
    },
    "data_protection": {
      "encryption": {
        "in_transit": true,
        "at_rest": true
      },
      "sensitive_data": {
        "api_keys": "encrypted",
        "credentials": "encrypted",
        "trade_data": "encrypted"
      }
    }
  },
  "performance": {
    "requirements": {
      "latency": {
        "analysis": "< 500ms",
        "execution": "< 100ms",
        "data_collection": "real-time"
      },
      "throughput": {
        "trades_per_second": 10,
        "analysis_per_second": 100
      },
      "resource_usage": {
        "cpu": "< 70%",
        "memory": "< 2GB",
        "disk": "< 80%"
      }
    },
    "optimization": {
      "caching": true,
      "connection_pooling": true,
      "batch_processing": true
    }
  },
  "monitoring": {
    "metrics": [
      "System health",
      "Trading performance",
      "Error rates",
      "Resource usage"
    ],
    "logging": {
      "levels": ["DEBUG", "INFO", "WARN", "ERROR"],
      "format": "JSON",
      "required_fields": [
        "timestamp",
        "level",
        "service",
        "message",
        "trace_id"
      ]
    },
    "alerts": {
      "conditions": [
        "Error rate > 1%",
        "Latency > 1s",
        "Memory > 80%",
        "Failed trades > 0"
      ]
    }
  },
  "documentation": {
    "required_docs": [
      "API documentation",
      "Architecture overview",
      "Setup guide",
      "Deployment guide",
      "Testing guide"
    ],
    "code_documentation": {
      "requirements": [
        "All public APIs documented",
        "Example usage provided",
        "Error cases documented",
        "Configuration documented"
      ]
    }
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oyi77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
