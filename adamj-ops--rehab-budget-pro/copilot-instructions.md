## budget-model

> The budget system uses three columns to track costs through the project lifecycle:

# Three-Column Budget Model

## Overview

The budget system uses three columns to track costs through the project lifecycle:

1. **Underwriting** (`underwriting_amount`)
   - Pre-deal estimate during acquisition analysis
   - Used for MAO calculations and deal evaluation
   - Set during initial project creation

2. **Forecast** (`forecast_amount`)
   - Post-walkthrough or contractor bid estimate
   - Updated after property inspection
   - More accurate than underwriting

3. **Actual** (`actual_amount`)
   - Real spend during/after construction
   - Updated as work is completed
   - Used for final profit calculations

## Variance Calculations

Computed columns in database (don't calculate client-side):

```sql
forecast_variance = forecast_amount - underwriting_amount
-- Positive = scope creep, Negative = under estimate

actual_variance = actual_amount - forecast_amount
-- Positive = over budget, Negative = under budget

total_variance = actual_amount - underwriting_amount
-- Overall variance from initial estimate
```

## Budget Flow

```
Project Creation → Underwriting amounts set (qty × rate)
     ↓
Property Walkthrough → Forecast amounts updated
     ↓
Construction → Actual amounts recorded
     ↓
Project Complete → Final variance analysis
```

## UI Display

### Budget Detail Tab
```
Category / Item          | Underwriting | Forecast | Actual | Variance
─────────────────────────────────────────────────────────────────────
Kitchen                  |    $12,000   |  $13,500 | $14,200|  +$2,200
  └─ Cabinets           |     $5,000   |   $5,500 |  $5,800|    +$800
  └─ Countertops        |     $4,000   |   $4,500 |  $4,900|    +$900
  └─ Appliances         |     $3,000   |   $3,500 |  $3,500|    +$500
```

### Color Coding
- **Green**: Under budget (negative variance)
- **Red**: Over budget (positive variance)
- **Gray**: No variance or no actual yet

### Summary Cards
```tsx
<div className="grid grid-cols-4 gap-4">
  <StatCard label="Underwriting" value={underwritingTotal} />
  <StatCard label="Forecast" value={forecastTotal} />
  <StatCard label="Actual" value={actualTotal} />
  <StatCard 
    label="Total Variance" 
    value={totalVariance}
    variant={totalVariance > 0 ? 'destructive' : 'success'}
  />
</div>
```

## MAO Calculation

Maximum Allowable Offer uses underwriting budget:

```sql
mao = arv * 0.7 - rehab_budget_with_contingency

-- Where:
rehab_budget = COALESCE(forecast_total, underwriting_total)
contingency_amount = rehab_budget * contingency_percent
rehab_budget_with_contingency = rehab_budget + contingency_amount
```

## Active Scenario Logic

The "active" budget phase depends on data availability:

```typescript
function getActiveScenario(project: ProjectSummary) {
  if (project.actual_total > 0) return 'actual';
  if (project.forecast_total > 0) return 'forecast';
  return 'underwriting';
}
```

## Profit/ROI by Scenario

Calculate profit for each budget phase:

```typescript
function calculateProfit(project: ProjectSummary, scenario: 'underwriting' | 'forecast' | 'actual') {
  const rehabBudget = {
    underwriting: project.underwriting_total,
    forecast: project.forecast_total || project.underwriting_total,
    actual: project.actual_total || project.forecast_total || project.underwriting_total,
  }[scenario];
  
  const contingency = rehabBudget * project.contingency_percent;
  const totalRehab = rehabBudget + contingency;
  const holdingCosts = project.holding_costs_monthly * project.hold_months;
  const sellingCosts = project.arv * project.selling_cost_percent;
  
  const totalInvestment = project.purchase_price + project.closing_costs + holdingCosts + totalRehab;
  const grossProfit = project.arv - sellingCosts - totalInvestment;
  const roi = (grossProfit / totalInvestment) * 100;
  
  return { grossProfit, roi, totalInvestment };
}
```

## Database Schema

```sql
-- budget_items table
underwriting_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
forecast_amount DECIMAL(12,2) NOT NULL DEFAULT 0,
actual_amount DECIMAL(12,2),

-- Generated columns (computed automatically)
forecast_variance DECIMAL(12,2) GENERATED ALWAYS AS (forecast_amount - underwriting_amount) STORED,
actual_variance DECIMAL(12,2) GENERATED ALWAYS AS (actual_amount - forecast_amount) STORED,
total_variance DECIMAL(12,2) GENERATED ALWAYS AS (actual_amount - underwriting_amount) STORED,
```

## Best Practices

1. **Always set underwriting first** - Required for MAO calculations
2. **Update forecast after walkthrough** - More accurate than initial estimates
3. **Record actual as work completes** - Don't wait until project end
4. **Use views for totals** - Never sum client-side
5. **Show all three columns** - Transparency in budget evolution

---
> Source: [adamj-ops/rehab-budget-pro](https://github.com/adamj-ops/rehab-budget-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
