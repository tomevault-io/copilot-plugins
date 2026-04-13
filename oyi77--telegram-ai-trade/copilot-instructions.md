## telegram-ai-trade

> Comprehensive rules for maintaining high code quality through style, testing, documentation, and best practices.

{
  "style_guide": {
    "python": {
      "base": "PEP 8",
      "additions": {
        "line_length": 88,
        "docstring_style": "Google",
        "import_order": [
          "Standard library",
          "Third party",
          "Local application"
        ],
        "naming": {
          "classes": "PascalCase",
          "functions": "snake_case",
          "variables": "snake_case",
          "constants": "UPPER_CASE",
          "private": "_prefix"
        }
      }
    },
    "typescript": {
      "base": "Airbnb",
      "additions": {
        "semicolons": "always",
        "quotes": "single",
        "interface_prefix": "I",
        "type_prefix": "T"
      }
    }
  },
  "testing": {
    "unit_tests": {
      "framework": "pytest",
      "coverage": {
        "minimum": "80%",
        "target": "90%",
        "critical_paths": "100%"
      },
      "requirements": [
        "Test each function",
        "Test edge cases",
        "Test error conditions",
        "Mock external dependencies"
      ],
      "naming": "test_<function_name>_<scenario>"
    },
    "integration_tests": {
      "coverage": {
        "api_endpoints": "100%",
        "data_flows": "100%",
        "critical_paths": "100%"
      },
      "requirements": [
        "Test component interaction",
        "Test data persistence",
        "Test error handling",
        "Test race conditions"
      ]
    },
    "performance_tests": {
      "requirements": [
        "Load testing",
        "Stress testing",
        "Endurance testing",
        "Spike testing"
      ],
      "metrics": [
        "Response time",
        "Throughput",
        "Error rate",
        "Resource usage"
      ]
    }
  },
  "documentation": {
    "code": {
      "requirements": [
        "Module documentation",
        "Class documentation",
        "Function documentation",
        "Complex logic explanation"
      ],
      "docstring_template": {
        "function": [
          "Short description",
          "Args:",
          "Returns:",
          "Raises:",
          "Example:"
        ],
        "class": [
          "Class description",
          "Attributes:",
          "Methods:",
          "Example:"
        ]
      }
    },
    "api": {
      "format": "OpenAPI/Swagger",
      "requirements": [
        "Endpoint description",
        "Request/response schema",
        "Error responses",
        "Authentication",
        "Rate limits"
      ]
    }
  },
  "code_organization": {
    "structure": {
      "module_size": "< 500 lines",
      "function_size": "< 30 lines",
      "class_size": "< 300 lines",
      "file_organization": [
        "Imports",
        "Constants",
        "Classes",
        "Functions",
        "Main"
      ]
    },
    "patterns": {
      "required": [
        "Dependency Injection",
        "Factory Pattern",
        "Strategy Pattern",
        "Observer Pattern"
      ],
      "avoided": [
        "Global state",
        "Tight coupling",
        "God objects",
        "Magic numbers"
      ]
    }
  },
  "error_handling": {
    "requirements": {
      "exceptions": {
        "custom_hierarchy": true,
        "meaningful_names": true,
        "proper_documentation": true
      },
      "logging": {
        "levels": {
          "debug": "Detailed information",
          "info": "Normal operations",
          "warning": "Potential issues",
          "error": "Runtime errors",
          "critical": "System failures"
        },
        "format": {
          "timestamp": true,
          "level": true,
          "module": true,
          "message": true,
          "stack_trace": "if error"
        }
      }
    }
  },
  "security": {
    "requirements": {
      "input_validation": true,
      "output_encoding": true,
      "authentication": true,
      "authorization": true,
      "data_protection": true
    },
    "practices": [
      "Secure by default",
      "Principle of least privilege",
      "Defense in depth",
      "Fail securely"
    ]
  },
  "performance": {
    "guidelines": {
      "algorithms": "O(n) or better",
      "database": {
        "queries": "Optimized",
        "indices": "Required",
        "connections": "Pooled"
      },
      "caching": {
        "strategy": "Required",
        "invalidation": "Documented"
      }
    },
    "monitoring": {
      "metrics": [
        "Response time",
        "Memory usage",
        "CPU usage",
        "I/O operations"
      ],
      "profiling": "Required for critical paths"
    }
  }
}

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oyi77) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
