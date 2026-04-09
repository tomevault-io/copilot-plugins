## edgar-parser

> **Last Updated:** 2025-12-11

# CLAUDE.MD - AI Assistant Context File

**Last Updated:** 2025-12-11
**Project:** EDGAR Financial Data Parser
**Status:** Active Development - Parser V2 Complete

---

## Project Overview

This project parses SEC EDGAR financial data to calculate **ROCE (Return on Capital Employed)** and **Earnings Yield** for ~374 S&P 500 companies using the Greenblatt Magic Formula methodology.

**Core Purpose:** Identify high-quality, undervalued companies by combining:
- **ROCE** - Measures business quality (higher is better)
- **Earnings Yield** - Measures valuation (higher = cheaper)

**Data Sources:**
- SEC EDGAR API (free) - Financial statements
- Wikipedia (free) - S&P 500 constituent list
- Yahoo Finance (free) - Market capitalization

**Target Universe:** ~374 S&P 500 companies (filtered to compatible sectors)

---

## Current State

### ✅ Completed
- [x] Parser V2 with modular calculator architecture
- [x] EBIT Calculator with 4-tier waterfall strategy
- [x] Debt Calculator with 3-component structure
- [x] Cash Calculator (unrestricted vs restricted)
- [x] Balance Sheet Calculator
- [x] Historical data extraction (all fiscal periods)
- [x] ROCE calculation with interpretation
- [x] Earnings Yield components (ready for market cap)
- [x] S&P 500 company list download script
- [x] Database schema design

### 🚧 In Progress
- [ ] Full S&P 500 parsing (~374 companies)
- [ ] Market cap integration (yfinance)
- [ ] Database loader for historical data
- [ ] Sector benchmark calculations

### 📋 Planned
- [ ] REST API with FastAPI
- [ ] Magic Formula screening tool
- [ ] Daily automated updates
- [ ] Web dashboard

---

## Key Architecture Decisions

### Parser V2 Modular Design

The parser uses **specialized calculators** for each metric type:

1. **EBIT Calculator** (`ebit_calculator.py`)
   - 4-tier waterfall with validation
   - Tier 1: Direct OperatingIncomeLoss OR validated Revenues - CostsAndExpenses
   - Tier 2: Revenues - COGS - OpEx
   - Tier 3: Net Income + Tax + Interest
   - Tier 4: Pre-tax Income + Interest

2. **Debt Calculator** (`debt_calculator.py`)
   - 3-component structure: LT non-current + LT current + ST borrowings
   - Double-counting prevention
   - Handles both 2-component and 3-component structures

3. **Cash Calculator** (`cash_calculator.py`)
   - Distinguishes unrestricted (for EV) vs total cash
   - Priority: Direct tag → Total - Restricted → Calculated

4. **Balance Sheet Calculator** (`balance_sheet_calculator.py`)
   - Extracts Assets and Current Liabilities
   - Calculates Capital Employed

**Why this matters:** Each calculator encapsulates complex logic and fallback strategies, making the code maintainable and testable.

### Data Flow

```
SEC EDGAR API
    ↓
Parser V2 → Specialized Calculators
    ↓
Extract metrics for ALL fiscal periods
    ↓
Calculate ROCE per period
    ↓
Prepare Earnings Yield components (needs market cap)
    ↓
PostgreSQL database (time-series)
    ↓
REST API
    ↓
Frontend dashboard
```

---

## Important Files

### Core Parser Files
- `src/edgar_parser/parser_v2.py` - Main parser (USE THIS, not parser.py)
- `src/edgar_parser/ebit_calculator.py` - EBIT extraction
- `src/edgar_parser/debt_calculator.py` - Debt extraction
- `src/edgar_parser/cash_calculator.py` - Cash extraction
- `src/edgar_parser/balance_sheet_calculator.py` - Assets/liabilities
- `src/edgar_parser/config.py` - Configuration and environment

### Scripts
- `scripts/build_company_list.py` - Download S&P 500 list from Wikipedia
- `scripts/test_updated_parser.py` - Test Parser V2
- `scripts/parse_all_companies.py` - Parse full S&P 500 dataset
- `scripts/test_db_connection.py` - Verify PostgreSQL connection
- `scripts/test_edgar_fetch.py` - Verify SEC API access

### Documentation
- `docs/00_START_HERE_MASTER_FOUNDATION.md` - Complete project vision and build plan
- `docs/EBIT_CALCULATION.md` - EBIT calculation details
- `README.md` - Quick start guide
- `PARSER_V2_README.md` - Parser V2 documentation

### Database
- `database/schema.sql` - PostgreSQL schema

### Test Data
- `data/apple_edgar_live.json` - Live EDGAR data for Apple (test file)
- `data/target_companies.csv` - S&P 500 company list (generated)
- `data/raw_edgar/` - Raw EDGAR JSON files
- `data/parsed/` - Parsed output files

---

## Key Formulas

### ROCE (Return on Capital Employed)
```
ROCE = (EBIT / Capital Employed) × 100

Where:
  Capital Employed = Total Assets - Current Liabilities
```

**Interpretation:**
- Excellent: > 25%
- Good: 15-25%
- Average: 10-15%
- Poor: < 10%

### Earnings Yield
```
Earnings Yield = (EBIT / Enterprise Value) × 100

Where:
  Enterprise Value = Market Cap + Net Debt
  Net Debt = Total Debt - Unrestricted Cash
```

**Note:** Market Cap must be fetched from external API (yfinance, Alpha Vantage, etc.)

---

## Common Tasks

### Test Parser on Single Company
```bash
python scripts/test_updated_parser.py
```

### Download S&P 500 Company List
```bash
python scripts/build_company_list.py
# Output: data/target_companies.csv (~374 companies)
```

### Parse All Companies
```bash
python scripts/parse_all_companies.py
# Takes ~12 minutes for 374 companies
```

### Test Database Connection
```bash
python scripts/test_db_connection.py
```

### Run Tests
```bash
pytest tests/
```

---

## Database Schema Overview

### Tables

**companies** - Company master data
- ticker, company_name, sector, cik
- ~374 rows (S&P 500 filtered)

**financial_metrics** - Raw metrics from EDGAR
- company_id, fiscal_year_end, filing_date, form
- ebit, total_debt, unrestricted_cash, total_assets, current_liabilities
- Time-series data (multiple rows per company)

**calculated_ratios** - Derived metrics
- company_id, calculation_date
- roce, earnings_yield, enterprise_value
- Time-series data

**market_data** - Market cap from external API
- company_id, date, market_cap, source
- Updated daily

**sector_benchmarks** - Industry averages
- sector, calculation_date
- median_roce, median_earnings_yield, company_count
- Updated weekly

---

## Known Issues and Gotchas

### EDGAR API Rate Limiting
- Limit: 10 requests/second
- Required: User-Agent header with real name/email
- Set in `.env`: `EDGAR_USER_AGENT="YourName your@email.com"`

### Sector Filtering
- **Exclude:** Financials (banks), Real Estate (REITs), Utilities (regulated)
- **Why:** Incompatible accounting (ROCE doesn't apply)
- **Result:** ~374 from 500 companies

### Parser V1 vs V2
- **DO NOT USE:** `src/edgar_parser/parser.py` (old)
- **USE:** `src/edgar_parser/parser_v2.py` (current)
- V2 is complete rewrite with modular calculators

### Data Consistency
- All metrics must be from **same fiscal period**
- Parser V2 enforces this by extracting by period
- Check `fiscal_year_end` field to ensure alignment

### Market Cap
- Parser V2 does NOT fetch market cap (by design)
- Market cap is volatile, needs daily updates
- Integrate separately with yfinance or similar

---

## Testing Approach

### Validation Companies
The parser has been validated against:
- **AAPL** (Apple) - Tech, high ROCE
- **CVX** (Chevron) - Energy, complex debt structure
- **CAT** (Caterpillar) - Industrials, multi-segment
- **XOM** (Exxon) - Energy, large scale

### What to Test
When making changes:
1. Run test script: `python scripts/test_updated_parser.py`
2. Verify ROCE calculation accuracy
3. Check all 4 EBIT tiers work
4. Validate debt doesn't double-count
5. Ensure unrestricted cash used (not total)

### Example Test Results (Apple 2023)
```
ROCE: 55.10% (Excellent)
EBIT: $114.3B
Total Debt: $105.1B
Unrestricted Cash: $61.6B
Net Debt: $43.5B
Capital Employed: $187.2B
```

---

## Environment Setup

### Required
- Python 3.9+
- PostgreSQL 14+
- Virtual environment (.venv)

### Environment Variables (.env)
```bash
# Database
DATABASE_URL=postgresql://user:password@localhost/edgar_financial_metrics

# SEC EDGAR API
EDGAR_USER_AGENT="YourName your@email.com"  # REQUIRED

# Optional
LOG_LEVEL=INFO
```

### Installation
```bash
# Create venv
python3 -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Setup database
make db-setup-all
```

---

## Code Style and Conventions

### When Writing Code
- Use type hints for function signatures
- Document complex logic with inline comments
- Follow existing naming conventions:
  - `snake_case` for functions and variables
  - `PascalCase` for classes
  - `UPPER_CASE` for constants

### When Modifying Calculators
- Each calculator should be **self-contained**
- Return structured dict with `value`, `method`, `sources`
- Include `warnings` list for data quality issues
- Use logging for debug info, not print statements

### When Adding Tests
- Place in `tests/` directory
- Name: `test_<feature>.py`
- Use pytest fixtures for common setup
- Include both success and failure cases

---

## AI Development Guidelines

### What Claude Should Know

1. **Parser V2 is the current version** - Don't use or reference parser.py
2. **Modularity is key** - Each calculator is independent
3. **Data quality matters** - Always validate and log warnings
4. **Historical data** - Extract all periods, not just latest
5. **Market cap is external** - Parser only prepares components

### When Asked to Make Changes

**DO:**
- Read existing code first before suggesting changes
- Test changes with `scripts/test_updated_parser.py`
- Update documentation if adding features
- Follow existing patterns in calculators
- Add logging for debugging

**DON'T:**
- Break the modular calculator structure
- Hard-code ticker-specific logic
- Ignore data validation
- Add unnecessary dependencies
- Over-engineer solutions

### Common Requests and How to Handle

**"Parse a new company"**
```python
from edgar_parser.parser_v2 import EDGARParser
import json

# Load EDGAR data
with open('data/raw_edgar/TICKER_edgar_raw.json') as f:
    edgar_data = json.load(f)

# Parse
parser = EDGARParser()
result = parser.parse_company_data(edgar_data, verbose=True)

# Check results
print(result['annual_periods'][0]['calculated_ratios']['roce'])
```

**"Fix EBIT calculation"**
1. Check `src/edgar_parser/ebit_calculator.py`
2. Review 4-tier waterfall logic
3. Test with multiple companies (not just one)
4. Verify validation logic still works

**"Add new metric"**
1. Create new calculator in `src/edgar_parser/`
2. Follow pattern: take `facts` dict, return structured result
3. Update `parser_v2.py` to call new calculator
4. Add to output schema
5. Test thoroughly

---

## Debugging Tips

### Parser Returns None for EBIT
- Check if company uses non-standard tags
- Review `ebit_calculator.py` tier logic
- Add company-specific tag to config if needed
- Check EDGAR data structure

### ROCE Seems Wrong
- Verify all metrics from same fiscal period
- Check Capital Employed calculation
- Review Current Liabilities vs Total Liabilities
- Compare to manual calculation

### Debt Double-Counting
- Check `debt_calculator.py` detection logic
- Verify 3-component vs 2-component structure
- Look for tags that include current maturities
- Review component breakdown in output

### Missing Historical Data
- Ensure using `parser_v2.py` (not old parser.py)
- Check EDGAR data has multiple periods
- Verify fiscal year grouping logic
- Review `annual_periods` vs `quarterly_periods`

---

## Resources

### External Documentation
- [SEC EDGAR API Docs](https://www.sec.gov/edgar/sec-api-documentation)
- [XBRL Tags Reference](https://www.sec.gov/info/edgar/edgartaxonomies)
- [Greenblatt Magic Formula](https://www.magicformulainvesting.com/)

### Internal Documentation
- Master Foundation: `docs/00_START_HERE_MASTER_FOUNDATION.md`
- EBIT Details: `docs/EBIT_CALCULATION.md`
- Parser V2 Guide: `PARSER_V2_README.md`

### Key Concepts
- **Capital Employed**: Assets - Current Liabilities (not Total Liabilities!)
- **Net Debt**: Total Debt - Unrestricted Cash (not total cash!)
- **Enterprise Value**: Market Cap + Net Debt
- **EBIT**: Operating income BEFORE interest and taxes

---

## Success Criteria

### For Parser V2 (✅ Complete)
- [x] Modular calculator architecture
- [x] 4-tier EBIT waterfall with validation
- [x] 3-component debt structure
- [x] Unrestricted cash distinction
- [x] Historical data extraction
- [x] Comprehensive logging
- [x] Tested on 4+ diverse companies

### For Full System (In Progress)
- [ ] Parse 350+ S&P 500 companies (94%+ success rate)
- [ ] Integrate market cap from yfinance
- [ ] Calculate sector benchmarks (50-77 companies each)
- [ ] Build REST API
- [ ] Daily automated updates
- [ ] Web dashboard

---

## Quick Reference

### File Naming Conventions
- Raw EDGAR data: `data/raw_edgar/{TICKER}_edgar_raw.json`
- Parsed output: `data/parsed/{TICKER}_parsed.json`
- Company list: `data/target_companies.csv`

### Tag Naming Patterns (XBRL)
- Assets: `Assets`, `AssetsCurrent`, `AssetsNoncurrent`
- Liabilities: `LiabilitiesCurrent`, `Liabilities`
- Debt: `LongTermDebt`, `ShortTermBorrowings`, `LongTermDebtCurrent`
- Cash: `CashAndCashEquivalentsAtCarryingValue`, `RestrictedCash`
- Income: `OperatingIncomeLoss`, `Revenues`, `NetIncomeLoss`

### Common CLI Commands
```bash
# Activate venv
source .venv/bin/activate

# Test parser
python scripts/test_updated_parser.py

# Build company list
python scripts/build_company_list.py

# Parse all companies
python scripts/parse_all_companies.py

# Database setup
make db-setup-all

# Run tests
pytest tests/
```

---

## Version History

**V2.0** (Current - December 2024)
- Complete refactor with modular calculators
- 4-tier EBIT waterfall
- 3-component debt structure
- Historical data support
- Comprehensive validation and logging

**V1.0** (Deprecated)
- Original monolithic parser
- Basic metric extraction
- No validation
- Single period only

---

## Contact / Support

This is a solo project by Cary Schwartzstein.

For issues or questions:
- Check documentation first
- Review test outputs
- Examine calculation logs in parsed JSON
- Test with known-good company (Apple)

---

**Remember:** The goal is to build a reliable, automated system for finding high-quality, undervalued companies using the Magic Formula. Quality and accuracy matter more than speed or features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/caryschwartzstein)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/caryschwartzstein)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
