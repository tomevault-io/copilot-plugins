## trading-control

> CI/CD pipeline requirements and common fixes for trading-control project


# CI/CD Patterns & Common Fixes

## Critical CI/CD Commands (Must Pass)

### Exact Pipeline Commands
```bash
# Step 1: Ruff linting (must show "All checks passed!")
ruff check . --fix

# Step 2: Ruff formatting (must show "X files already formatted") 
ruff format --check .

# Step 3: Critical error checks (must show "All checks passed!")
ruff check . --select=E9,F63,F7,F82

# Step 4: Test suite (all tests must pass)
pytest tests/ -v --tb=short
```

## Common CI/CD Failure Patterns & Fixes

### B008: Function Call in Default Argument
```python
# ❌ WRONG - B008 error
async def analyze_trade(request, trading_service=Depends(get_trading_service)):

# ✅ RIGHT - Use Annotated syntax
from typing import Annotated
async def analyze_trade(
    request, 
    trading_service: Annotated[TradingService, Depends(get_trading_service)]
):
```

### F821: Undefined Name (Missing Import)
```python
# ❌ WRONG - F821 error
trading_service: Annotated[TradingService, Depends(get_trading_service)]

# ✅ RIGHT - Add missing import
from api.services.trading_service import TradingService
trading_service: Annotated[TradingService, Depends(get_trading_service)]
```

### B904: Raise Without From
```python
# ❌ WRONG - B904 error
except Exception as e:
    raise HTTPException(status_code=500, detail=str(e))

# ✅ RIGHT - Add exception chaining
except Exception as e:
    raise HTTPException(status_code=500, detail=str(e)) from None
```

## Schema Version Compliance

### Database Write Requirements
```python
# All new writes must include schema_version
await writer.write(
    table="agent_runs",
    data={
        "strategy_id": strategy_id,
        "symbol": "BTC/USD",
        "action": "buy",
        "schema_version": "v3",  # MANDATORY
        "source": "reasoning_agent"
    }
)
```

## Test Requirements

### Test File Naming
```bash
# Test files must follow pattern
tests/test_{module_name}.py
tests/agents/test_{agent_name}.py
tests/api/test_{router_name}.py
```

### Test Coverage Requirements
- Every agent: `tests/agents/test_{agent_name}.py`
- Every API router: `tests/api/test_{router_name}.py`
- Bug fixes: Add regression test that would have caught the bug

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SamuelMatthew95) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
