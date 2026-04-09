## crowdfunding

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Python-based crowdfunding project management system for real estate investments. It tracks expenses, payments, partners, and sales for a crowdfunding project called "Ghadeer Land". The system calculates ownership percentages, payment statuses, and project balances.

## Commands

### Installing Dependencies
```bash
pip install -r requirements.txt
```

### Running the Application

#### Default Mode (Hardcoded Ghadeer Land Project)
```bash
python main.py
```

#### Configuration File Mode (Recommended)
```bash
# Run with YAML configuration file
python main.py --config projects/ghadeer_land.yaml

# Show all available command line options
python main.py --help
```

### Reporting Options

#### Show Specific Reports
```bash
# Show only partner summary
python main.py --config projects/ghadeer_land.yaml --partners

# Show only expenses chronologically
python main.py --config projects/ghadeer_land.yaml --expenses-by-date

# Show only payments chronologically
python main.py --config projects/ghadeer_land.yaml --payments-by-date

# Show only sales summary with ownership percentages
python main.py --config projects/test_with_sales.yaml --sales

# Show sales chronologically
python main.py --config projects/test_with_sales.yaml --sales-by-date

# Show only project summary
python main.py --config projects/ghadeer_land.yaml --summary

# Show all available reports (adapts to what's in your config)
python main.py --config projects/my_project.yaml --all
```

#### Date Filtering
```bash
# Show expenses since June 1, 2025
python main.py --config projects/ghadeer_land.yaml --expenses-by-date --since 2025-06-01

# Show payments since August 1, 2025
python main.py --config projects/ghadeer_land.yaml --payments-by-date --since 2025-08-01

# Combine multiple reports with date filter
python main.py --config projects/ghadeer_land.yaml --expenses-by-date --payments-by-date --since 2025-07-01
```

#### Configuration Validation
```bash
# Validate configuration file without running reports
python main.py --config projects/ghadeer_land.yaml --validate-only
```

## Configuration System

### YAML Configuration Format

The system supports configurable projects via YAML files. This allows easy creation and management of multiple projects without hardcoding data.

#### Configuration File Structure

**Required Section (Project Information Only):**
```yaml
project:
  name: "Project Name"
  start_date: "YYYY-MM-DD"
  end_date: "YYYY-MM-DD"
```

**All Optional Sections:**
```yaml
partners:  # Optional - can have projects with no partners defined yet
  - name: "Partner Name"
    investment_amount: 100000.00
    # OR use mathematical expressions:
    # investment_amount: "50000 + 30000 + 20000"

expenses:  # Optional - can have projects with no expenses defined yet
  - description: "Expense Description"
    amount: 50000.00
    # OR use mathematical expressions:
    # amount: "25000 + 15000 + 10000"
    date: "YYYY-MM-DD"

payments:  # Optional - can have projects with no payments yet
  # Partner payment (from partner investment)
  - amount: 50000.00
    # OR use mathematical expressions:
    # amount: "25000 + 25000"
    date: "YYYY-MM-DD"
    partner: "Partner Name"
    expense: "Expense Description"

  # Sales reinvestment (payment from sales revenue)
  - amount: 30000.00
    date: "YYYY-MM-DD"
    expense: "Another Expense"
    from_sales: true  # No partner field when from_sales is true

sales:     # Optional - new section for revenue tracking
  - description: "Property Sale Description"
    amount: 1500000.00
    # OR use mathematical expressions:
    # amount: "1200000 + 300000"
    date: "YYYY-MM-DD"
```

#### Mathematical Expressions

The system supports mathematical expressions in amount fields using addition (+) operations:

**Supported Formats:**
- Simple numbers: `100000`, `1500.50`
- Addition expressions: `"100 + 200 + 300"`
- Mixed decimals: `"1000.50 + 500.25"`
- Flexible spacing: `"100+200"`, `"100 + 200"`, `"  100  +  200  "`

**Examples:**
```yaml
partners:
  - name: "Ali AlDawood"
    # Break down complex calculations for clarity
    investment_amount: "205481 + 221.60 + 703 + 400 + 920 + 394"

expenses:
  - description: "Total Construction Cost"
    amount: "50000 + 30000 + 20000"  # 100000

payments:
  - amount: "25000 + 25000"  # 50000
    partner: "Partner Name"
    expense: "Expense Description"

sales:
  - description: "Property Unit A Sale"
    amount: "1200000 + 300000"  # 1500000
    date: "2025-06-15"
```

**Security Notes:**
- Only addition (+) operations are supported for security
- Only numbers, decimal points, plus signs, and spaces are allowed
- Expressions are validated before evaluation
- Invalid expressions will cause clear error messages

#### Creating New Projects

**Flexible Project Configuration:**
You can create projects at any stage of development - from initial planning (project info only) to complete projects with all sections.

**Basic Steps:**
1. **Start minimal**: Create a project with only project information
2. **Add sections as needed**: Partners, expenses, payments, and sales are all optional
3. **Use expressions**: Break down complex calculations using mathematical expressions
4. **Validate**: Use `--validate-only` flag to check for errors
5. **Run reports**: Use `--config path/to/your/project.yaml` to run reports

**Configuration Examples:**

```yaml
# Minimal project (planning stage)
project:
  name: "New Development Project"
  start_date: "2025-01-01"
  end_date: "2025-12-31"
```

```yaml
# Fundraising stage (partners defined)
project:
  name: "Fundraising Project"
  start_date: "2025-01-01"
  end_date: "2025-12-31"

partners:
  - name: "Lead Investor"
    investment_amount: "500000 + 300000"
  - name: "Secondary Investor"
    investment_amount: 250000
```

```yaml
# Active project with sales but no detailed expenses
project:
  name: "Revenue-Generating Project"
  start_date: "2025-01-01"
  end_date: "2025-12-31"

partners:
  - name: "Investor A"
    investment_amount: 1000000

sales:
  - description: "Property Sale Unit 1"
    amount: "1500000 + 300000"
    date: "2025-09-15"
```

#### Configuration Validation

The system validates:
- Required fields are present (only project section is mandatory)
- Date formats are correct (YYYY-MM-DD)
- Investment, expense, payment, and sales amounts are positive numbers
- Mathematical expressions are valid and secure (addition only)
- Partner and expense references in payments exist (when applicable)
- No duplicate partner names, expense descriptions, or sale descriptions
- Optional sections can be omitted entirely
- Payments with `from_sales: true` do not have a `partner` field
- Payments with `from_sales: true` reference a valid expense
- Total payments to an expense do not exceed the expense amount (over-payment prevention)
- Chronological warnings if reinvestments are dated before sales exist

#### Sales Calculations and Ownership

When sales are included in your configuration, the system automatically calculates profit distribution based on partner investment percentages:

**Ownership Calculation:**
- Each partner's ownership = (their investment / total investments) × 100%
- Ownership percentages are used to distribute sales revenue fairly

**Sales Output Example:**
```
Sale: Property Unit A Sale
Date: 2025-06-15
Total: SAR 1,500,000.00
------------------------------

Partner: Lead Investor
Amount based on ownership percentage of 55.56%: SAR 833,333.33
------------------------------
Partner: Co-Investor
Amount based on ownership percentage of 27.78%: SAR 416,666.67
```

**Key Features:**
- Automatic percentage calculation based on investment amounts
- Mathematical expressions supported in sales amounts
- Clear breakdown of each partner's share
- SMS-friendly formatting maintained

#### Sales Reinvestment Feature

The system supports **sales reinvestment**, allowing you to pay for project expenses using revenue from sales instead of (or in addition to) partner investments.

**How It Works:**
- Payments can come from two sources: partner investments or sales revenue
- When paying from sales revenue, use `from_sales: true` in the payment configuration
- Sales reinvestments reduce the **net sales** available for distribution to partners
- Partner ownership percentages remain unchanged (based on original investments)
- Net sales can become negative if reinvestments exceed sales revenue (creating debt)

**YAML Configuration:**
```yaml
payments:
  # Regular partner payment
  - amount: 50000
    date: "2025-03-15"
    partner: "Ali AlDawood"
    expense: "Land Purchase"

  # Payment from sales revenue (reinvestment)
  - amount: 30000
    date: "2025-06-20"
    expense: "Construction Costs"
    from_sales: true  # No partner field required

  # Mixed: same expense paid by both partner and sales
  - amount: 20000
    date: "2025-07-01"
    partner: "Mohammed Hassan"
    expense: "Construction Costs"
```

**Key Calculations:**
- **Gross Sales** = Sum of all sales revenue
- **Sales Reinvestments** = Sum of payments where `from_sales: true`
- **Net Sales** = Gross Sales - Sales Reinvestments
- **Partner Distribution** = Net Sales × Partner Ownership %

**Validation Rules:**
- ✅ Payments with `from_sales: true` **cannot** have a `partner` field
- ✅ Payments with `from_sales: true` **must** reference a valid expense
- ✅ Total payments to an expense cannot exceed the expense amount (over-payment prevention)
- ⚠️ Warning issued if reinvestment is dated before any sales exist
- ✅ Mathematical expressions supported in reinvestment amounts

**Example Scenario:**
```yaml
sales:
  - description: "Property Sale Unit A"
    amount: 500000
    date: "2025-06-01"

payments:
  # Reinvest 100,000 from sales into new construction
  - amount: 100000
    date: "2025-06-15"
    expense: "Phase 2 Construction"
    from_sales: true

# Result:
# Gross Sales: 500,000 SAR
# Reinvestments: 100,000 SAR
# Net Sales: 400,000 SAR
# Each partner receives their % of 400,000 SAR (not 500,000)
```

**Zero/Negative Sales Scenarios:**
- If no sales exist but reinvestments are configured:
  - Gross Sales = 0
  - Net Sales = 0 - Reinvestments = Negative (debt)
  - Partners' shares will be negative (liability)
- If reinvestments exceed sales revenue:
  - Net Sales = Negative (debt scenario)
  - Clearly indicated in reports with warning icons

**Report Displays:**
- **Sales Report**: Shows gross sales, itemized reinvestments, and net sales for distribution
- **Expense Report**: Shows payment source breakdown (partners vs. sales)
- **Partner Report**: Shows share of net sales (can be negative)
- **Payments Report**: Clearly indicates if payment is from sales revenue
- **Summary Report**: Displays gross sales, reinvestments, and net sales separately

## Architecture

### Core Components

- **CrowdfundingProject** (`src/crowdfunding_project.py`): Main orchestrator class that manages all project aspects including expenses, payments, partners, and sales. Provides comprehensive reporting and calculation methods.

- **Partner** (`src/partner.py`): Represents investors with their investment amounts. Used to calculate ownership percentages.

- **Expense** (`src/expense.py`): Represents project costs that need to be covered by partner payments.

- **Payment** (`src/payment.py`): Links partners to expenses, tracking who paid what toward which expense.

- **Sale** (`src/sale.py`): Represents revenue from property sales.

- **PaymentStatus** (`src/payment_status.py`): Enum for tracking expense payment completion (UNPAID, PARTIALLY_PAID, FULLY_PAID).

- **Helpers** (`src/helpers.py`): Utility functions for formatting output displays.

- **ConfigLoader** (`src/config_loader.py`): Loads and validates YAML configuration files with comprehensive error handling.

- **ProjectFactory** (`src/project_factory.py`): Creates CrowdfundingProject instances from YAML configuration files.

- **ExpressionEvaluator** (`src/expression_evaluator.py`): Safely evaluates mathematical expressions in configuration values, supporting addition operations with comprehensive validation.

### Data Flow

#### Configuration-Based Flow (Recommended)
1. The main application (`main.py`) parses command-line arguments
2. ConfigLoader loads and validates the YAML configuration file
3. ProjectFactory creates a CrowdfundingProject instance from the configuration
4. Partners, expenses, and payments are automatically added from configuration data
5. The system calculates ownership percentages based on investment amounts
6. Selected reports are displayed based on command-line flags

#### Legacy Flow (Hardcoded Data)
1. The main application (`main.py`) imports hardcoded project data
2. Expenses are added to define project costs
3. Partners are added with their investment commitments
4. Payments link partners to specific expenses they're covering
5. The system calculates ownership percentages based on investment amounts
6. Various summary reports show project status, partner positions, and expense tracking

### Key Business Logic

- **Ownership Calculation**: Based on partner investment amounts as percentage of total investments
- **Payment Tracking**: Associates payments with specific expenses to track completion status
- **Balance Calculation**: Revenue (sales) minus payments to show project profitability
- **Investment Balance**: Shows remaining committed investment per partner after payments made

## Project Structure

```
src/
├── crowdfunding_project.py  # Main project management class
├── partner.py              # Partner/investor representation
├── expense.py              # Project expense tracking
├── payment.py              # Payment linking partners to expenses
├── sale.py                 # Revenue tracking
├── payment_status.py       # Payment completion status enum
├── helpers.py              # Display formatting utilities
├── config_loader.py        # YAML configuration file loader and validator
└── project_factory.py      # Creates projects from configuration files

projects/
├── ghadeer_land.yaml       # Ghadeer Land project configuration file
└── ghadeer_land/           # Legacy hardcoded project data
    ├── __init__.py
    ├── partners.py
    ├── expenses.py
    └── payments.py
```

The main application logic is in `main.py` which supports both configuration-driven projects (recommended) and legacy hardcoded project data.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/geohussain)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/geohussain)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
