## khaata

> Project: Personal Finance Tracker with CSV Import

Project: Personal Finance Tracker with CSV Import

1. Overview

A web application to manage personal finances by importing CSV bank statements, mapping columns, categorizing transactions, and viewing financial reports (P&L and Balance Sheet).

2. Tech Stack

Frontend: Next.js 14 (App Router), TypeScript, Tailwind CSS.
UI Components: Shadcn UI (Radix Primitives).
Database: Supabase (PostgreSQL).
CSV Parsing: PapaParse.
State Management: React Hooks / Context.

3. Database Schema (Supabase)
   Table: accounts
   id: uuid (PK)
   user_id: uuid (FK to auth.users)
   name: text (e.g., "Chase Checking", "Amex Gold")
   type: text (Enum: 'ASSET', 'LIABILITY')
   opening_balance: decimal (Required for Balance Sheet accuracy)
   created_at: timestamp

Table: categories
id: uuid (PK)
user_id: uuid
name: text (e.g., "Food", "Utilities")
type: text (Enum: 'INCOME', 'COGS', 'EXPENSE', 'TRANSFER')

Table: subcategories
id: uuid (PK)
category_id: uuid (FK to categories)
name: text (e.g., "Groceries", "Restaurants")

Table: transactions
id: uuid (PK)
account_id: uuid (FK to accounts)
date: date
description: text
amount: decimal (Positive for deposits, Negative for withdrawals)
category_id: uuid (FK to categories, nullable)
subcategory_id: uuid (FK to subcategories, nullable)
auto_categorized: boolean (Default: false. If true, transaction was categorized using AI and needs manual approval)

4. Core Features & Logic

A. Account Management
User can create accounts.
Critical: User MUST input an opening_balance date and amount when creating an account.
Logic: Current Balance = Opening Balance + Sum of all transactions.

B. CSV Import & Mapping
User uploads a CSV file for a specific account.
UI: Show a "Mapping Screen" where user maps CSV headers to DB fields:
Date column -> transactions.date
Description column -> transactions.description
Amount column -> transactions.amount (Handle credit/debit columns if necessary)
Logic: Parse using PapaParse, preview data, then bulk insert into Supabase.

C. Categorization

A "Transactions" view (Table).
Inline dropdowns to assign Category and Subcategory to transactions.
"Bulk Edit" feature to categorize multiple selected rows at once.
When a user manually updates a transaction's category/subcategory, set auto_categorized to false (indicating manual approval).
Transactions with auto_categorized = true are marked as "AI - Needs Approval" in the UI and excluded from P&L calculations.

D. Reports

1. Profit & Loss (P&L) Statement

Timeframe: Monthly (e.g., "Jan 2024").

Logic:

- Query MUST exclude transactions where `category.type` is 'TRANSFER'.
- Query MUST exclude transactions where `auto_categorized` is true (AI-categorized transactions need manual approval before being included in P&L).
- Transfers are balance sheet movements, not profit/loss.
- AI-categorized transactions are excluded until manually approved (auto_categorized set to false).
  Total Income = Sum of transactions where category.type = 'INCOME' AND auto_categorized = false.
  Total COGS = Sum of transactions where category.type = 'COGS' AND auto_categorized = false.
  Gross Profit = Total Income - Total COGS.
  Total Operating Expenses = Sum of transactions where category.type = 'EXPENSE' AND auto_categorized = false.
  Net Profit = Gross Profit - Total Operating Expenses.
  View: Group by Category -> Subcategory.

2. Balance Sheet

Timeframe: "As of [Date]".

Logic:
Assets = Sum of (Opening Balance + Transactions) for all accounts with type 'ASSET'.
Liabilities = Sum of (Opening Balance + Transactions) for all accounts with type 'LIABILITY'.
Equity = Assets - Liabilities (This should match the cumulative Net Profit over time + Initial Equity).

Design:

- Use Dark theme.
- Use Tailwind CSS for styling.
- Use Shadcn UI for components.

Notes:

- Supabase credentials are in the .env.local file.
- DB schema is in the schema.sql file.
- Avoid displaying currency anywhere in the UI.
- For displaying datetime, use format dd/mm/yyyy. Dont keep month first. date first. In any format.

---
> Source: [kakshilshah/khaata](https://github.com/kakshilshah/khaata) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
