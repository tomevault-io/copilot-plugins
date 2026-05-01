## cw

> Provides SQLite database operations for financial data including:

# AGENTS.md - Oxidation Finance V2.0 Development Guide

**⚠️ CRITICAL: READ THIS FILE FIRST**  
This is the project's "black box" - the single source of truth for context restoration.

---
## 🧠 PROJECT MEMORY CORE

### Project Identity
- **Name**: CWZS (氧化加工财务系统)
- **Root**: `E:\python\CWZS`
- **Main Version**: v2.0 (`oxidation_finance_v20/`)
- **Active Development**: YES
- **Last Major Refactor**: 2025-02 (Service layer + security hardening)

### Known Issues (Active)

| File | Line(s) | Issue Type | Fix Status | Description |
|------|---------|------------|-------------|-------------|
| `tests/test_web_api.py` | 23 | ImportError | ⚠️ Open | `ModuleNotFoundError: No module named 'oxidation_finance_v20.web_app'` - web_app missing |
| `examples/generate_comprehensive_demo.py` | 20 | Import Error | ✅ Fixed | Incorrect `sys.path` causing `ModuleNotFoundError` (fixed to `parent.parent.parent`) |
| `tools/setup_wizard.py` | 193, 319, 335 | Bare except | ✅ Fixed | Replaced `except:` with `except sqlite3.Error:` |
| `tools/smart_calculator.py` | 338 | Bare except | ✅ Fixed | Replaced `except:` with `except Exception:` |
| `tools/quick_panel.py` | 270 | Bare except | ✅ Fixed | Replaced `except:` with `except Exception:` |
| `tools/data_quality_check.py` | 246 | Bare except | ✅ Fixed | Replaced `except:` with `except sqlite3.Error:` |
| `database/schema.py` | 240 | SQL Injection | ✅ Fixed | f-string DROP TABLE replaced with allowlist validation |
| `tools/setup_wizard.py` | 178, 288 | SQL Injection | ✅ Fixed | f-string SELECT replaced with `.format()` + allowlist |
| `tools/backup_restore.py` | 236 | SQL Injection | ✅ Fixed | f-string SELECT replaced with `.format()` + allowlist |

**Legacy Files** (not in main version):
- Redundant test runners in root: ~15 `run_*.py` and `verify_*.py` scripts (consider consolidating)
- Old version directories moved to `deprecated_versions/`

### Code Quality Status

| Category | Status | Notes |
|----------|--------|------|
| Type Safety | ✅ 95% | Decimal/UUID/Optional properly used |
| SQL Injection | ✅ Fixed | All table names validated via allowlist |
| Error Handling | ✅ Improved | Bare except clauses eliminated |
| Service Layer | ✅ Complete | `services/__init__.py` implements separation |
| Test Coverage | ✅ 70+ core tests passing | Database, order, user, accrual all green |
| Demo Generator | ✅ Fixed | `examples/generate_comprehensive_demo.py` runs successfully |
| Quick Panel | ✅ Enhanced | Friendly error message for missing database |

### Recent Commits (Top 5)
```bash
acd99de refactor(project-structure): reorganize repository and improve documentation
829888b security: fix SQL injection vulnerabilities with table name allowlists
2456c03 feat: add WebService layer for improved separation of concerns
dfa1c25 fix: critical type safety and None-check errors in FinanceManager
088d0f0 fix: resolve critical test failures and improve web UI
```

### Current State
- **Service Layer**: ✅ Implemented in `services/__init__.py`
- **SQL Injection**: ✅ Fixed (table allowlists added)
- **Type Safety**: ✅ Mostly compliant (Decimal, UUID, Optional)
- **Test Coverage**: 426 tests passing (excluding web_api import issue)
- **Web Layer**: ⚠️ `web_app.py` not using service layer (needs integration)

### AI Collaboration Protocol

**CRITICAL RULES:**
1. **Always start here** - Read this file before any work
2. **Update this file** after any significant change
3. **Document decisions** in the "Known Issues" and "Recent Context" sections

**Workflow:**
- Phase 0: Read AGENTS.md → Understand context
- Phase 1: Run `pytest` → Check current state
- Phase 2: Create todo list → Get approval (if complex)
- Phase 3: Implement → Verify tests pass
- Phase 4: Update AGENTS.md → Commit with clear message

**When committing:**
```bash
git add <changed files>
git commit -m "type(scope): brief description

Detailed explanation if needed (wrap at 72 chars)"
```

**DO NOT:**
- Delete old versions without documenting in AGENTS.md
- Merge branches without updating this file
- Skip test runs before committing

---

## Project Overview

**Oxidation Finance V2.0** is a financial management system for small oxidation processing enterprises, built with Python 3.8+. The main development directory is `oxidation_finance_v20/`.

## Build/Lint/Test Commands

### Running Tests

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_database.py

# Run specific test class
pytest tests/test_database.py::TestDatabaseBasics

# Run specific test
pytest tests/test_database.py::TestDatabaseBasics::test_customer_crud

# Run with coverage
pytest --cov=oxidation_finance_v20

# Run specific marker
pytest -m unit          # Unit tests only
pytest -m integration   # Integration tests
pytest -m property      # Property-based tests
pytest -m database      # Database tests
pytest -m slow          # Slow tests (use sparingly)

# Run with hypothesis (property-based)
pytest tests/ --hypothesis-show-statistics
```

### Quick Test Scripts

```bash
# Quick test runners (located in oxidation_finance_v20/)
python quick_test.py              # Basic quick test
python run_accrual_tests.py      # Accrual tests
python run_audit_tests.py        # Audit tests
python run_backup_tests.py        # Backup/restore tests
python verify_*.py               # Verification scripts
```

### Dependencies

```bash
# Install dependencies
pip install -r requirements.txt

# V2.0 specific requirements (in oxidation_finance_v20/)
pip install -r oxidation_finance_v20/requirements.txt
```

## Code Style Guidelines

### File Headers

Every Python file must include:

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
Module description here.
"""
```

### Imports

**Order:**
1. Standard library imports
2. Third-party imports
3. Local application imports

**Example:**

```python
from dataclasses import dataclass, field
from datetime import date, datetime
from decimal import Decimal
from enum import Enum
from typing import List, Optional, Dict, Any
import uuid

import pytest
import pandas as pd

from oxidation_finance_v20.database import DatabaseManager
from oxidation_finance_v20.models import Customer, ProcessingOrder
```

**Local imports always use absolute paths from project root, never relative imports.**

### Data Models (Dataclasses)

Use `@dataclass` for all business models.

```python
@dataclass
class Customer:
    """Customer information"""
    id: str = field(default_factory=lambda: str(uuid.uuid4()))
    name: str = ""
    contact: str = ""
    phone: str = ""
    credit_limit: Decimal = Decimal("0")
    created_at: datetime = field(default_factory=datetime.now)

    def is_credit_available(self, amount: Decimal) -> bool:
        """Check if credit limit allows this amount"""
        return amount <= self.credit_limit
```

**Rules:**
- Use `Decimal` for all monetary values (NOT float)
- Use `Optional[T]` for nullable fields
- Use `field(default_factory=...)` for mutable defaults (list, dict, datetime)
- Use descriptive field comments for business context
- Include docstrings for all classes and public methods

### Enums

Use Enum for type-safe constants with Chinese descriptions:

```python
class OrderStatus(Enum):
    """Order status"""
    PENDING = "待加工"
    IN_PROGRESS = "加工中"
    COMPLETED = "已完工"
    DELIVERED = "已交付"
    PAID = "已收款"
```

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Classes | PascalCase | `Customer`, `OrderManager` |
| Functions | snake_case | `get_customer()`, `calculate_total()` |
| Variables | snake_case | `customer_id`, `total_amount` |
| Constants | UPPER_SNAKE_CASE | `DEFAULT_PAGE_SIZE`, `MAX_RETRY_COUNT` |
| Private methods | _snake_case | `_validate_input()`, `_get_connection()` |
| Private attributes | _snake_case | `_db_path`, `_config_dir` |
| Type variables | PascalCase | `T = TypeVar('T')` |

### Error Handling

**DO:**
```python
def get_customer(self, customer_id: str) -> Optional[Customer]:
    """Get customer by ID"""
    try:
        cursor = self.conn.cursor()
        cursor.execute("SELECT * FROM customers WHERE id = ?", (customer_id,))
        row = cursor.fetchone()
        if row is None:
            return None
        return self._row_to_customer(row)
    except sqlite3.Error as e:
        logger.error(f"Database error getting customer {customer_id}: {e}")
        raise DatabaseError(f"Failed to retrieve customer: {e}") from e
```

**DON'T:**
- Never use bare `except:` clauses
- Never suppress errors with `pass`
- Never return `None` silently on failure

**Custom exceptions** should be defined in appropriate modules:

```python
class DatabaseError(Exception):
    """Base database error"""
    pass

class ValidationError(Exception):
    """Input validation error"""
    pass
```

### Type Hints

Always use type hints for function signatures:

```python
def calculate_profit(
    self,
    income: Decimal,
    expenses: List[Decimal],
    tax_rate: float = 0.15
) -> Tuple[Decimal, Decimal, Decimal]:
    """Calculate profit, tax, and net income"""
    ...
```

### String Formatting

Use f-strings for all string formatting:

```python
name = "客户"
result = f"订单号: {order.order_no}, 金额: {order.total_amount}"
```

### Database Patterns

**Connection management:**

```python
class DatabaseManager:
    def __init__(self, db_path: str):
        self.db_path = db_path
        self.conn = None

    def connect(self):
        self.conn = sqlite3.connect(self.db_path)
        self.conn.row_factory = sqlite3.Row

    def close(self):
        if self.conn:
            self.conn.close()
```

**Context manager pattern:**

```python
# In tests or when automatic cleanup is needed
with DatabaseManager("finance.db") as db:
    customer = db.get_customer(customer_id)
```

**Parameterize all queries:**

```python
# CORRECT
cursor.execute("SELECT * FROM customers WHERE id = ?", (customer_id,))

# WRONG - SQL injection risk
cursor.execute(f"SELECT * FROM customers WHERE id = {customer_id}")
```

### Logging

Use the logging module with consistent formatting:

```python
import logging

logger = logging.getLogger(__name__)

def save_order(self, order: ProcessingOrder) -> str:
    """Save processing order"""
    logger.info(f"Saving order: {order.order_no}")
    try:
        # ... save logic
        logger.info(f"Order saved successfully: {order.id}")
        return order.id
    except Exception as e:
        logger.error(f"Failed to save order {order.order_no}: {e}")
        raise
```

### Testing Patterns

**Use pytest fixtures from conftest.py:**

```python
def test_order_crud(self, temp_db, sample_customer, sample_order):
    """Test order CRUD operations"""
    temp_db.save_customer(sample_customer)
    order_id = temp_db.save_order(sample_order)
    assert order_id == sample_order.id
```

**Use Decimal for monetary assertions:**

```python
from decimal import Decimal

assert order.total_amount == Decimal("550.00")
# NOT: assert order.total_amount == 550.00
```

**Naming conventions:**
- Test files: `test_*.py`
- Test classes: `Test*` (CamelCase)
- Test methods: `test_*` (snake_case)

**Docstrings for tests:**

```python
def test_customer_credit_limit(self, temp_db):
    """
    Test that customer credit limits are enforced correctly.

    This verifies:
    - Orders within credit limit are allowed
    - Orders exceeding credit limit are rejected
    """
    ...
```

### Configuration

Use JSON files for user configurations:

```json
{
  "version": "2.0.0",
  "reminder": {
    "tax_reminder_days": [7, 3, 1, 0],
    "payable_reminder_days": 3
  },
  "storage": {
    "data_dir": "data",
    "backup_dir": "backups"
  }
}
```

Load with:

```python
import json
from pathlib import Path

def load_config(config_path: Path) -> dict:
    """Load configuration from JSON file"""
    with open(config_path, 'r', encoding='utf-8') as f:
        return json.load(f)
```

### File Structure

```
oxidation_finance_v20/
├── models/              # Data models (dataclasses, enums)
├── database/            # DB schema and managers
├── business/            # Business logic layer
├── reports/             # Report generation
├── config/              # Configuration management
├── utils/               # Utility functions
├── tests/               # Pytest tests and fixtures
├── examples/            # Example scripts
├── scripts/             # Build/deployment scripts
├── requirements.txt     # Dependencies
├── pytest.ini          # Pytest configuration
└── README.md           # Documentation
```

### Documentation

**Module docstrings:**

```python
"""
Oxidation Finance V2.0 - Database Manager

Provides SQLite database operations for financial data including:
- Customer management
- Order processing
- Income/expense tracking
- Bank transaction management
"""
```

**Method docstrings (Google style):**

```python
def save_order(self, order: ProcessingOrder) -> str:
    """
    Save a processing order to the database.

    Args:
        order: ProcessingOrder instance to save

    Returns:
        The ID of the saved order

    Raises:
        DatabaseError: If save operation fails
        ValidationError: If order data is invalid

    Example:
        >>> order = ProcessingOrder(order_no="OX202401001", ...)
        >>> order_id = db.save_order(order)
    """
```

### Commit Messages

Follow conventional commits:

- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation
- `test:` Adding tests
- `refactor:` Code restructuring
- `chore:` Maintenance tasks

Example:
```
feat(database): add customer credit limit validation
fix: resolve order total calculation error
docs: update API documentation for v2.0
```

### Key Business Rules

1. **Monetary values**: Always use `Decimal`, never `float`
2. **IDs**: Use `uuid.uuid4()` for unique identifiers
3. **Dates**: Use `date` for dates, `datetime` for timestamps
4. **Bank types**: G银行 (formal transactions with invoices), N银行 (WeChat/cash)
5. **计价单位**: 件, 条, 只, 个, 米, 公斤, 平方米
6. **加工工序**: 喷砂 → 拉丝 → 抛光 → 氧化

## Business Domain Knowledge

### Pricing Units (计价方式)

| Unit | Chinese | Description |
|------|---------|-------------|
| PIECE | 件 | Small parts, screws |
| STRIP | 条 | Long strips, aluminum bars |
| UNIT | 只 | Individual items, handles |
| ITEM | 个 | Components, assemblies |
| METER | 米 | Length-based, aluminum profiles |
| KILOGRAM | 公斤 | Weight-based, plates |
| SQUARE_METER | 平方米 | Area-based, panels |

### Process Types (加工工序)

| Process | Chinese | Notes |
|---------|---------|-------|
| SANDBLASTING | 喷砂 | Often outsourced |
| WIRE_DRAWING | 拉丝 | Sometimes outsourced |
| POLISHING | 抛光 | Sometimes outsourced |
| OXIDATION | 氧化 | Final process, never outsourced |

### Expense Types (支出类型)

| Expense Type | Chinese | Notes |
|--------------|---------|-------|
| RENT | 房租 | Monthly factory rent |
| UTILITIES | 水电费 | Electricity and water |
| ACID_THREE | 三酸 | Sulfuric, nitric, hydrochloric acid |
| CAUSTIC_SODA | 片碱 | Sodium hydroxide |
| SODIUM_SULFITE | 亚钠 | Sodium sulfite |
| COLOR_POWDER | 色粉 | Coloring powder for oxidation |
| DEGREASER | 除油剂 | Degreasing agent |
| FIXTURES | 挂具 | Hanging fixtures for oxidation |
| OUTSOURCING | 外发加工费 | Outsourcing processing fees |
| DAILY_EXPENSE | 日常费用 | Daily expenses |
| SALARY | 工资 | Employee salaries |
| OTHER | 其他 | Other expenses |

### Bank Account Types

| Bank Type | Chinese | Usage |
|-----------|---------|-------|
| G_BANK | G银行 | Formal transactions WITH invoices |
| N_BANK | N银行 | Cash/WeChat transactions, NO invoices |

### Key Business Scenarios

**Scenario 1: Order with Outsourcing**
```
Order Amount: ¥5,000
- Customer: XXX Company
- Items: 100 meters of aluminum profile
- Processes: Sandblasting + Oxidation
- Outsourced: Sandblasting (¥1,000)
- Profit: ¥4,000 (after outsourcing cost)
```

**Scenario 2: Income Allocation**
```
Customer Payment: ¥10,000 (one payment)
- Order A: ¥4,000
- Order B: ¥3,000
- Order C: ¥3,000
(Single income record with allocation to multiple orders)
```

**Scenario 3: Pre-payment (预收款)**
```
Customer pays: ¥10,000 (before order)
- Not yet assigned to specific order
- Recorded as "pre-payment"
- Later allocated when order is created
```

### Demo Data

Generate comprehensive demo data:

```bash
python oxidation_finance_v20/examples/generate_comprehensive_demo.py
```

This creates:
- 6 customers (various pricing units)
- 6 suppliers (raw materials + outsourcing)
- 45 orders (90 days of data)
- 52 income records (including pre-payments)
- 95 expense records (all expense types)
- 2 bank accounts (G + N bank)
- 147 bank transactions

---
## 📊 Current State & Quick Reference

### Active Development Status
- **Main Branch**: `master`
- **Service Layer**: ✅ Complete (`services/__init__.py`)
- **Security**: ✅ SQL injection fixed (table allowlists)
- **Type Safety**: ✅ 95% compliant
- **Test Suite**: ✅ 426 tests passing

### Quick Diagnostic Commands
```bash
# Check project status
cat AGENTS.md

# Run full test suite
cd oxidation_finance_v20 && pytest

# Run specific test modules
pytest tests/test_database.py -v
pytest tests/test_finance_manager.py -v
pytest tests/test_order_manager.py -v

# Check for issues
pytest --tb=short 2>&1 | grep -E "(FAILED|ERROR)"

# Create demo data
python oxidation_finance_v20/examples/generate_comprehensive_demo.py

# Start web interface
python oxidation_finance_v20/web_app.py
```

### Key File Locations
| Purpose | Path |
|---------|------|
| DB Schema | `oxidation_finance_v20/database/schema.py` |
| Service Layer | `oxidation_finance_v20/services/__init__.py` |
| Web Interface | `oxidation_finance_v20/web_app.py` |
| Models | `oxidation_finance_v20/models/business_models.py` |
| Tests | `oxidation_finance_v20/tests/` |
| Quick Tools | `oxidation_finance_v20/tools/` |
| Examples | `oxidation_finance_v20/examples/` |
| Config | `oxidation_finance_v20/config/` |

### Environment Requirements
- Python 3.8+
- Flask (for web interface)
- pytest (for testing)
- All dependencies in `requirements.txt`

---

---
> Source: [tel9980/cw](https://github.com/tel9980/cw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
