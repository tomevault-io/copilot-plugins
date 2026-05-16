## expense-autopilot

> An automated expense receipt management system with Claude Code integration.

# Expense Autopilot

An automated expense receipt management system with Claude Code integration.

## Project Overview

This project helps automate the tedious process of managing expense receipts for
filing in expense systems like Concur. It provides a unified `/expense` command
with multiple subcommands for different tasks.

## Main Command

Use `/expense` followed by an action and arguments:

```bash
/expense scan receipt.pdf
/expense process receipt.pdf
/expense batch
/expense report
/expense mileage "client meeting" 45
/expense check
/expense export concur
```

## Directory Structure

```text
expense-autopilot/
├── .claude/commands/expense/   # All expense commands
├── config/                     # Configuration files
│   └── policy.md              # Your company expense policy
├── receipts/                   # Receipt storage
│   ├── inbox/                 # New unprocessed receipts
│   ├── processed/             # Processed and renamed receipts
│   └── archive/               # Archived receipts
├── output/                     # Generated outputs
│   ├── reports/               # Expense reports
│   └── renamed/               # Renamed receipt copies
├── templates/                  # Narrative templates
└── logs/                      # Processing logs
```

## Configuration

1. Edit `config/policy.md` with your company's expense policy
2. Update `config/settings.json` with your specific limits and rules
3. Customize narrative templates in `templates/`

## File Naming Convention

`YYYY-MM-DD_CATEGORY_VENDOR_AMOUNT.pdf`

Example: `2025-07-22_MEALS_STARBUCKS_12.50.pdf`

---
> Source: [veteranbv/expense-autopilot](https://github.com/veteranbv/expense-autopilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
