## freight-ops

> This is an AI-powered freight operations platform designed specifically for hotshot trucking businesses. The platform leverages AI agents to automate and optimize critical freight operations including dispatch, rate analysis, compliance tracking, and route optimization.

# Freight Operations Platform - Claude Code Instructions

## Project Overview

This is an AI-powered freight operations platform designed specifically for hotshot trucking businesses. The platform leverages AI agents to automate and optimize critical freight operations including dispatch, rate analysis, compliance tracking, and route optimization.

**Current Status**: Early-stage development. Core data models and configuration are in place; agents and tools are planned but not yet implemented.

## Current Codebase Structure

```
freight-ops/
├── src/
│   ├── __init__.py              # Package init (version 0.1.0)
│   ├── agents/
│   │   └── __init__.py          # Placeholder - agents not yet implemented
│   ├── core/
│   │   └── __init__.py          # Placeholder - infrastructure not yet implemented
│   ├── data/
│   │   ├── __init__.py
│   │   └── models/
│   │       ├── __init__.py
│   │       └── load.py          # ✅ IMPLEMENTED - Core Load model with profitability calculations
│   └── tools/
│       └── __init__.py          # Placeholder - MCP tools not yet implemented
├── config/
│   ├── config.yaml              # Business configuration (H-4 Strategic Solutions)
│   └── llms.json                # LLM provider and agent assignment configuration
├── notebooks/
│   └── getting_started.ipynb    # Introductory notebook
├── scripts/
│   └── init_project.py          # Project initialization script
├── tests/
│   └── __init__.py              # Tests not yet implemented
├── .env.example                 # Environment variable template (comprehensive)
├── pyproject.toml               # Project dependencies and tool configuration
└── README.md                    # Project documentation
```

### What's Implemented

1. **`src/data/models/load.py`** - Fully implemented Load Pydantic model including:
   - Load status/type enums
   - Location model
   - Core Load model with all freight fields
   - Computed properties: `total_miles`, `deadhead_percentage`, `gross_revenue`, `rate_per_mile`, `all_miles_rate`, `trip_duration_hours`
   - Profitability check method: `is_profitable(min_rpm, max_deadhead_pct)`

2. **Configuration** - Fully set up:
   - `config/config.yaml` - Business rules for H-4 Strategic Solutions LLC (DOT# 4486526, MC# MC-1772833-C)
   - `config/llms.json` - LLM assignments per agent with fallback models
   - `.env.example` - All required API keys and settings documented

3. **Project tooling** - Configured in `pyproject.toml`:
   - Dependencies: anthropic, openai, langchain, mcp, pydantic, httpx, pandas, sqlalchemy, etc.
   - Dev tools: pytest, ruff, mypy, black
   - Entry points defined for CLI and agent commands

### What Needs Implementation

**Priority Order:**

1. **Base Agent Class** (`src/agents/base.py`)
   - MCP protocol integration
   - Tool registry access
   - Structured logging
   - Error handling patterns

2. **Core Infrastructure** (`src/core/`)
   - `config.py` - Configuration loader using dynaconf
   - `mcp_registry.py` - Tool discovery and management
   - `sandbox.py` - Secure execution environment (optional)

3. **Data Models** (`src/data/models/`) - Extend from Load model:
   - `route.py` - Route with waypoints, distances, timing
   - `rate.py` - Rate analysis and market data
   - `driver.py` - Driver info and equipment
   - `expense.py` - Fuel, tolls, maintenance tracking
   - `settlement.py` - Driver pay calculations

4. **AI Agents** (`src/agents/`)
   - `dispatch.py` - Load matching and booking
   - `rate_analysis.py` - Market rate analysis
   - `compliance.py` - IFTA, HOS, DOT compliance
   - `route_optimizer.py` - Deadhead reduction
   - `settlement.py` - Driver pay calculations

5. **MCP Tools** (`src/tools/`)
   - Load board integrations (DAT, Truckstop.com)
   - Mapping/routing tools
   - Financial calculators

## Development Guidelines

### Code Style and Quality

```bash
# Install dependencies
uv sync

# Install dev dependencies
uv sync --extra dev

# Lint and format
ruff check src/
ruff format src/

# Type checking
mypy src/

# Run tests
pytest tests/

# Run tests with coverage
pytest --cov=src tests/
```

### Python Standards

- **Python 3.12+** required
- Use **type hints everywhere** - this domain is complex, types help
- Use **Pydantic v2** for all data models (see `load.py` for patterns)
- Follow the **computed_field** pattern for derived values
- Use **Decimal** for all monetary values (never float)
- Use **Enum** classes for fixed choices (LoadStatus, LoadType)

### Adding New Data Models

Follow the pattern in `src/data/models/load.py`:

```python
from decimal import Decimal
from datetime import datetime
from typing import Optional
from pydantic import BaseModel, Field, computed_field

class MyModel(BaseModel):
    """Docstring explaining the model."""

    # Required fields with Field() for validation/description
    required_field: str = Field(..., description="What this field represents")

    # Optional fields with defaults
    optional_field: Optional[str] = None

    # Numeric validation
    positive_value: Decimal = Field(..., gt=0, description="Must be positive")

    @computed_field
    @property
    def derived_value(self) -> Decimal:
        """Computed property that appears in serialization."""
        return self.positive_value * 2
```

### Adding New Agents

Agents should follow this pattern (when base class is implemented):

```python
from src.agents.base import BaseAgent
from src.data.models import Load

class DispatchAgent(BaseAgent):
    """Dispatch agent for load matching and booking."""

    # Reference config/llms.json for model configuration
    agent_type = "dispatch"

    async def evaluate_load(self, load: Load) -> dict:
        """Evaluate a load for profitability."""
        # Log reasoning for human oversight
        self.log_decision(
            data_considered={"load_id": load.load_id, "rpm": load.rate_per_mile},
            thresholds_applied={"min_rpm": self.config.min_rpm},
            decision="accept" if load.is_profitable(...) else "reject",
            confidence=0.85
        )
```

### Configuration Access

Use the business config from `config/config.yaml`:

```python
# Access configuration values
company_name = config.company.name  # "H-4 Strategic Solutions LLC"
min_rpm = config.rates.minimum_rate_per_mile  # 2.00
max_deadhead = config.rates.max_deadhead_percentage  # 20
home_base = config.home_base  # Fort Gibson, OK
```

### LLM Model Selection

Reference `config/llms.json` for agent-specific models:

- **Dispatch**: Claude 3.5 Sonnet (temp 0.2) - analytical reasoning
- **Rate Analysis**: Claude 3.5 Sonnet (temp 0.1) - precise calculations
- **Compliance**: Claude 3.5 Sonnet (temp 0.0) - maximum accuracy
- **Route Optimizer**: Claude 3.5 Sonnet (temp 0.3) - creative solutions
- **Settlement**: Claude 3.5 Sonnet (temp 0.0) - financial accuracy
- **Monitoring**: Claude 3.5 Haiku - fast, low-cost
- **General Query**: Claude 3.5 Haiku - efficient

## Freight Domain Knowledge

### Key Concepts

**Rate Per Mile (RPM)**
- Gross revenue divided by loaded miles
- Minimum target for H-4: $2.00/mile
- Target: $2.50/mile
- Premium: $3.00/mile
- Always include fuel surcharge and accessorials

**Deadhead**
- Empty miles driven to pickup or after delivery
- Maximum threshold for H-4: 20% or 150 miles
- Calculate: `(deadhead_miles / total_miles) * 100`
- Critical for profitability - a high-paying load with long deadhead may be worse than lower-paying nearby load

**IFTA (International Fuel Tax Agreement)**
- Track fuel purchases and miles by jurisdiction
- Required for quarterly reporting
- Home state for H-4: Oklahoma
- Penalties for non-compliance are severe

**Driver Settlement**
- Calculate based on percentage or per-mile rate
- Deduct advances, fuel card purchases, insurance
- Must comply with owner-operator agreements

**Hotshot Trucking Specifics**
- Lighter loads (typically Class 3-5 trucks)
- Focus on speed and flexibility over maximum payload
- H-4 equipment: 2025 Dodge Ram 3500, 40ft gooseneck flatbed
- Max payload: 22,500 lbs (air-ride gooseneck — premium selling point for damage-sensitive freight)
- Preferred operating area: OK, TX, AR, KS, MO, LA (1000 mile radius)

### Operating Costs (H-4 Configuration)

- Fuel cost: $3.50/gallon
- MPG: 9
- Cost per mile: $0.39
- Maintenance reserve: $0.10/mile
- Insurance: $1,500/month
- Truck payment: $1,200/month
- Trailer payment: $400/month

## API Integrations

### Load Boards

- **DAT**: Primary load board, RateView for market rates
- **Truckstop.com**: Secondary load board
- Both require authentication and have rate limits
- Implement exponential backoff for API failures
- Cache frequently accessed data

### Required Environment Variables

Critical ones (see `.env.example` for full list):
- `ANTHROPIC_API_KEY` - Primary LLM provider
- `OPENAI_API_KEY` - Fallback LLM provider
- `DAT_API_KEY`, `DAT_API_SECRET` - DAT Load Board
- `TRUCKSTOP_API_KEY` - Truckstop.com
- `GOOGLE_MAPS_API_KEY` - Route optimization
- `DATABASE_URL` - PostgreSQL connection

## Testing Guidelines

```bash
# Run all tests
pytest tests/

# Run with verbose output
pytest -v tests/

# Run specific test file
pytest tests/agents/test_dispatch.py

# Run tests matching pattern
pytest -k "test_load"
```

### Test Patterns

- Use mock data for load board responses (avoid API costs)
- Test edge cases: no available loads, extreme rates, multi-stop routes
- Validate compliance calculations against known correct values
- Test agent decision-making with historical load data
- Use pytest markers: `@pytest.mark.unit`, `@pytest.mark.integration`, `@pytest.mark.api`

## Best Practices

1. **Always validate load data** - Brokers sometimes post incorrect info
2. **Consider total trip profitability** - Not just loaded miles
3. **Factor in realistic transit times** - Speed limits, HOS rules, loading times
4. **Implement circuit breakers** - Don't hammer APIs when they're down
5. **Log all agent decisions** - Critical for debugging and improvement
6. **Use type hints everywhere** - This is a complex domain, types help
7. **Document business logic** - Rate calculations, compliance rules, etc.
8. **Avoid over-engineering** - Keep solutions simple and focused

## Known Issues

1. **`config/config.yaml`** has extraneous content starting at line 80 (appears to be copy-pasted instructions that should be removed)

## Integration with open-ptc-agent

This project is designed to work alongside open-ptc-agent for advanced agent capabilities:
- Use MCP protocol for tool standardization
- Share context through structured data models
- Coordinate multi-agent workflows for complex operations
- Compatible with Claude Desktop and other MCP clients

---
> Source: [grantrb95/freight-ops](https://github.com/grantrb95/freight-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
