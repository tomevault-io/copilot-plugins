## database

> - Main schema: `supabase/schema.sql`

# Database Rules - Supabase/PostgreSQL

## Schema Location
- Main schema: `supabase/schema.sql`
- Migrations: `supabase/migrations/`
- Seed data: `supabase/seed.sql`

## Core Tables

### projects
Property details, financials, status tracking
```sql
-- Key fields
id, user_id, name, address, city, state, zip
arv, purchase_price, closing_costs, holding_costs_monthly, hold_months
selling_cost_percent, contingency_percent
status (enum: lead, analyzing, under_contract, in_rehab, listed, sold, dead)
```

### budget_items
Three-column budget model with computed variances
```sql
-- Key fields
id, project_id, vendor_id, category, item, description
qty, unit, rate
underwriting_amount, forecast_amount, actual_amount
forecast_variance, actual_variance, total_variance (GENERATED)
cost_type, status, priority, sort_order
```

### vendors
Global vendor directory (not project-specific)
```sql
-- Key fields
id, user_id, name, trade, contact_name, phone, email
licensed, insured, w9_on_file
rating (1-5), reliability, price_level, status
```

### draws
Payment tracking with milestones
```sql
-- Key fields
id, project_id, vendor_id, draw_number, milestone
amount, status (pending, approved, paid)
date_requested, date_paid, payment_method, reference_number
```

### line_item_photos
Photo storage metadata
```sql
-- Storage path format: project-photos/{project_id}/{line_item_id}/{uuid}.{ext}
id, line_item_id, project_id, storage_path, photo_type, caption
```

## Important Views (USE THESE FOR READS)

### project_summary
Projects with all calculated totals - ALWAYS use this for project queries
```sql
SELECT * FROM project_summary WHERE id = $1;
-- Includes: underwriting_total, forecast_total, actual_total
-- rehab_budget, rehab_actual, contingency_amount
-- total_investment, gross_profit, mao, roi
```

### budget_by_category
Category aggregates with variances
```sql
SELECT * FROM budget_by_category WHERE project_id = $1;
-- Includes: underwriting_total, forecast_total, actual_total per category
-- completed_count, in_progress_count, not_started_count
```

### vendor_payment_summary
Vendor totals across projects
```sql
SELECT * FROM vendor_payment_summary;
-- Includes: projects_count, total_paid, pending_amount
```

## Key Calculations (Handled by Views)

### Budget Item Amount
```sql
underwriting_amount = qty * rate (or manual entry)
forecast_variance = forecast_amount - underwriting_amount
actual_variance = actual_amount - forecast_amount
total_variance = actual_amount - underwriting_amount
```

### Project Financials
```sql
rehab_budget = COALESCE(forecast_total, underwriting_total)
contingency_amount = rehab_budget * contingency_percent
rehab_budget_with_contingency = rehab_budget + contingency_amount
selling_costs = arv * selling_cost_percent
holding_costs_total = holding_costs_monthly * hold_months
total_investment = purchase_price + closing_costs + holding_costs_total + rehab_budget_with_contingency
gross_profit = arv - selling_costs - total_investment
mao = arv * 0.7 - rehab_budget_with_contingency  -- 70% rule
```

## RLS Policies

All tables have Row Level Security:
- Users can only see/modify their own data
- Filter by `user_id = auth.uid()`

**Note**: Authentication not yet implemented - `user_id` is currently null in inserts.

## Migration Best Practices

1. Always create new migration files (don't modify existing)
2. Name format: `YYYYMMDDHHMMSS_description.sql`
3. Include both UP and DOWN (commented) migrations
4. Test in Supabase SQL Editor before committing

## Supabase Storage

Bucket: `project-photos`
- Max file size: 10MB
- Accepted types: jpg, png, webp, pdf
- Path format: `{project_id}/{line_item_id}/{uuid}.{ext}`
- Use signed URLs for secure access

---
> Source: [adamj-ops/rehab-budget-pro](https://github.com/adamj-ops/rehab-budget-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
