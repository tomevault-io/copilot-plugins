## taxsnax

> US tax preparation CLI - transforms bank statements and financial data into CPA-ready tax packets with 1099 tracking, expense categorization, and duplicate detection.

# TaxSnax — Agent Skill

## What
US tax preparation CLI - transforms bank statements and financial data into CPA-ready tax packets with 1099 tracking, expense categorization, and duplicate detection.

**Security Note:** TaxSnax intentionally does NOT store SSN, EIN, or sensitive PII. Collect W-9 forms separately and provide SSN/EIN directly to your CPA.

## When to Use
- "Prepare my taxes"
- "Import bank statements"
- "Track 1099 contractors"
- "Generate CPA packet"
- "Estimate quarterly taxes"
- "Categorize expenses"

## Quick Start
```bash
# Check status
taxsnax status -d .taxsnax

# Import statements
taxsnax import statements.csv --source "Chase" -d .taxsnax

# Generate CPA packet
taxsnax cpa-packet --year 2025 -o packet.txt -d .taxsnax
```

## Key Commands
```bash
# All commands support --json + -d for programmatic use
taxsnax init --year 2025 --json -d .taxsnax
taxsnax import <file> --source "Bank" --json -d .taxsnax
taxsnax summary --year 2025 --json -d .taxsnax
taxsnax contractors --json -d .taxsnax
taxsnax 1099-checklist --json -d .taxsnax
taxsnax cpa-packet --year 2025 --json -d .taxsnax
taxsnax tax-estimate --year 2025 --state PA --json -d .taxsnax
taxsnax reports --year 2025 -o ./reports --json -d .taxsnax
```

## Security First

### What TaxSnax Stores
- Transaction data (date, description, amount, category)
- Contractor names and payment totals
- Business name, address, type
- Personal name, address, filing status

### What TaxSnax Does NOT Store
- ❌ SSN (Social Security Number)
- ❌ EIN (Employer Identification Number)
- ❌ Bank account numbers
- ❌ Credit card numbers

### Best Practices
1. Collect W-9 forms separately from contractors
2. Store W-9s securely outside TaxSnax
3. Provide SSN/EIN to CPA directly, not through TaxSnax
4. Use TaxSnax for transaction organization only

## Integration Points
- Feeds into: CPA review, tax filing software
- Feeds from: Bank exports, credit card statements, accounting software

---
> Source: [quarlwithcode/taxsnax](https://github.com/quarlwithcode/taxsnax) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
