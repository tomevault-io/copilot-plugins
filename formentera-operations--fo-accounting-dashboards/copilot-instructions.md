## fo-accounting-dashboards

> This repo contains Claude.ai skill definitions that allow the Formentera Operations accounting team to query Snowflake gold models and generate Excel reports — replacing manual Quorum ODA downloads and pivot workflows.

# FO Accounting Dashboards — Developer Guide

This repo contains Claude.ai skill definitions that allow the Formentera Operations accounting team to query Snowflake gold models and generate Excel reports — replacing manual Quorum ODA downloads and pivot workflows.

## Before You Write a Skill

1. **Read the project README** for the report you're building → `projects/{domain}/README.md`
2. **Understand the gold model grain and filters** — see the Model Catalog below
3. **Check the analytics repo** (`Formentera-Operations/analytics`) for the actual SQL if you need column-level detail

## Snowflake Connection

Skills query the **production gold layer** in Snowflake:

| Setting | Value |
|---------|-------|
| Account | `YL35090.south-central-us.azure` |
| Database | `FO_PRODUCTION_DB` |
| Role | `DBT_ROLE` |
| Auth | RSA keypair |

**MCP tool:** Use the Snowflake MCP server configured in the Claude.ai Project. Never hardcode credentials in skill definitions.

## Gold Model Catalog

All gold models live in `FO_PRODUCTION_DB`. Schema names follow the pattern `gold_{domain}`.

---

### Accounts Receivable — `gold_financial`

#### `gold_fct_ar_aging_summary`
**ODA equivalent:** Report ARR (AR Report)
**Grain:** One row per invoice
**Standard filter:** `WHERE include_record = 1` for actionable outstanding items

| Column | Description |
|--------|-------------|
| `invoice_id` | PK |
| `invoice_number` | Invoice identifier |
| `company_code` / `company_name` | Operating company |
| `owner_id` / `owner_code` / `owner_name` | Invoice recipient |
| `well_id` / `well_code` / `well_name` | Associated well/property |
| `invoice_date` | Invoice date (used to calculate aging) |
| `days_past_due` | Days since invoice_date; negative = future-dated (not yet due) |
| `total_invoice_amount` | Gross invoice amount |
| `remaining_balance` | Outstanding balance (used in aging buckets) |
| `current_balance` | Balance not yet due (days_past_due <= 0) |
| `balance_1_30_days` | 1-30 days past due |
| `balance_31_60_days` | 31-60 days past due |
| `balance_61_90_days` | 61-90 days past due |
| `balance_90_plus_days` | Over 90 days past due |
| `is_invoice_posted` | `true` = posted to GL |
| `include_record` | `1` = include in standard aging report |
| `hold_billing` | Invoice is on billing hold |
| `voucher_number` | GL voucher reference |
| `invoice_description` | Free-text description |

**Common filter patterns:**

Standard AR aging (matches ODA ARR report): `WHERE include_record = 1`

Posted invoices only: `WHERE is_invoice_posted = true AND include_record = 1`

Pre-JIB preview (unposted): `WHERE is_invoice_posted = false`

---

#### `gold_fct_ar_aging_detail`
**ODA equivalent:** Report 640 (AR Detail Report)
**Grain:** One row per AR transaction (invoice, payment, adjustment, netting)

| Column | Description |
|--------|-------------|
| `invoice_id` | Parent invoice FK |
| `transaction_source` | `invoice` / `payment` / `adjustment` / `netting` |
| `transaction_date` | Date of the transaction |
| `transaction_type` | Invoice type label |
| `transaction_reference` | Free-text reference |
| `balance_due` | Transaction amount |
| `remaining_balance` | Parent invoice outstanding balance |
| `include_record` | `1` = include in standard report |
| `company_code` / `company_name` | Operating company |
| `owner_code` / `owner_name` | Invoice recipient |
| `well_code` / `well_name` | Associated well |
| `invoice_number` | Parent invoice number |
| `voucher_number` | GL voucher reference |
| `is_invoice_posted` | Posted status of parent invoice |
| `sort_order` | Display ordering |

---

### Revenue — `gold_financial`

#### `gold_fct_owner_revenue_detail`
**ODA equivalent:** Report ORD (Owner Revenue Detail)
**Grain:** One row per owner revenue distribution line item
**Filter:** Redistributed items (payment_status_id = 8) excluded at model build time — no runtime filter needed

| Column | Description |
|--------|-------------|
| `id` | Natural key (ODA distribution ID) |
| `owner_revenue_detail_sk` | Surrogate key |
| `company_code` | Operating company |
| `owner_id` / `owner_code` / `owner_name` | Owner |
| `fid` | Owner federal tax ID |
| `well_code` / `well_name` | Associated well |
| `production_date` / `production_year` / `production_month` | Production period |
| `product_code` / `product_name` / `product_full_name` | Product (oil, gas, NGL, etc.) |
| `decimal_interest` | Owner decimal interest |
| `interest_type_full_name` | WI, ORRI, NPI, etc. |
| `net_volume` | Net production volume |
| `net_value` | Gross revenue value |
| `paid_amount` | Amount paid to owner |
| `total_tax` | Severance and production taxes |
| `net_deductions` | Net deductions |
| `amount_suspended` | Amount held in suspense |
| `netted_amount` | Amount netted against AR |
| `balance_amount` | Final balance |
| `item_suspense` | Suspense reason at line item level |
| `owner_suspense` | Owner-level default suspense category |
| `well_suspense` | Well-level suspend revenue type |

---

### General Ledger — `gold_financial`

#### `gold_fct_gl_details`
**ODA equivalent:** AR Cross-Clear report, GL Detail Extract (GLE)
**Grain:** One row per GL journal entry line
**Used for:** AR Cross-Clear skill — no dedicated gold model needed, query directly with filters

| Column | Description |
|--------|-------------|
| `main_account` | GL main account code (e.g. 501, 130) |
| `sub_account` | GL sub account code (e.g. 1, 2, 3, 4) |
| `account_name` | GL account name (e.g. REVENUE PAYABLE, A/R JIB) |
| `voucher_code` | Voucher number — use to pair debit/credit rows |
| `entity_type` | Type of entity (owner, vendor, well, etc.) |
| `entity_code` | Entity identifier |
| `entity_name` | Entity display name |
| `journal_date` | Transaction date |
| `accrual_date` | Accounting period date |
| `gl_description` | Transaction description |
| `net_amount` | Amount — debits positive, credits negative |

**AR Cross-Clear filter pattern:**
```sql
WHERE main_account IN ('501', '130')
  AND sub_account IN ('1', '2', '3', '4')
  AND (
    gl_description LIKE '%Net Revenue Against A/R%'
    OR gl_description LIKE '%AR Cross Clear%'
  )
```

**Pairing logic:** Each cross-clear voucher produces two rows that net to zero —
one debit (501.x Revenue Payable) and one credit (130.x A/R JIB). Group by
`voucher_code` to see matched pairs.

---

### Accounts Payable — `gold_financial`

#### `gold_dim_ap_check_register`
**ODA equivalent:** Report 330 (AP Check Register)
**Grain:** One row per AP check

| Column | Description |
|--------|-------------|
| `company_code` / `company_name` | Operating company |
| `check_number` | Check / transaction number |
| `check_date` | Date issued |
| `check_type` | Payment type code |
| `check_type_name` | Full payment type label (System/Manual + type) |
| `check_amount` | Payment amount |
| `main_account` / `sub_account` / `account_name` | GL account |
| `voucher_code` | GL voucher reference |
| `entity_code` / `entity_name` | Payee (vendor entity) |
| `void_date` | Date voided (null if not voided) |
| `reconciled` | `YES` if reconciled, blank if not |
| `voided` | `YES` if voided, `NO` if not |

---

### Dimensions

| Model | Schema | Description |
|-------|--------|-------------|
| `gold_dim_well` | `gold_asset_hierarchy` | Well master — API#, name, status, operator |
| `gold_dim_owner` | `gold_land` | Owner master — name, address, interest types |
| `gold_dim_vendor` | `gold_supply_chain` | Vendor master |
| `gold_dim_company` | `gold_organization` | Company master |
| `gold_dim_entity` | `gold_organization` | Entity master (shared across vendors/owners) |
| `gold_dim_purchaser` | `gold_marketing` | Purchaser master for revenue |
| `gold_dim_afe` | `gold_financial` | AFE master |

---

## Skill Structure

Each report domain lives in `projects/{domain}/skills/`. Each skill is a Markdown file defining the Claude prompt, expected inputs, query pattern, and output format.

```
projects/
  ar_aging/
    skills/
      ar_aging_summary.md     <- structured report skill (Report ARR)
      ar_detail_drilldown.md  <- entity drill-down skill (Report 640)
    docs/
      field_mapping.md        <- ODA column -> gold model column mapping
      business_logic.md       <- filter patterns, edge cases, gotchas
```

### Skill File Template

```markdown
# Skill: {Report Name}

## Purpose
[What accounting question this answers]

## ODA Equivalent
Report {code} - {name}

## Gold Models Used
- `FO_PRODUCTION_DB.{schema}.{model}` - {why}

## Input Parameters
- `period` (required): Accounting period, e.g. "February 2026"
- `company` (optional): Company code filter

## Standard Filters
[Document the WHERE clause pattern for standard report output]

## Output Columns
[Ordered list matching the ODA report layout]

## Excel Format
[Column widths, header row, summary totals, freeze panes]

## Edge Cases
[Known gotchas - pre-JIB invoices, voided checks, suspense categories]
```

## How to Build a Skill

1. **Find the ODA report** in the mapping table → `docs/brainstorms/2026-02-26-oda-reports-claude-skills-brainstorm.md`
2. **Identify the gold model** — confirm grain, key columns, and standard filters from this CLAUDE.md
3. **Check the analytics repo** for the actual SQL if column behavior is unclear
4. **Write the skill file** using the template above
5. **Test the query** against `FO_PRODUCTION_DB` using the Snowflake MCP before finalizing
6. **Update the project README** to mark the skill as complete

## Key Business Concepts

| Term | Meaning |
|------|---------|
| `include_record = 1` | Include in standard AR aging — excludes paired advance/closeout transactions |
| `is_invoice_posted` | Invoice has been posted to the GL; unposted = pre-JIB or draft |
| `hold_billing` | Invoice is on billing hold — typically excluded from aging reports |
| `payment_status_id = 8` | Revenue redistributed — excluded from owner revenue detail at model build time |
| `days_past_due` | Negative = future-dated invoice (not yet due); zero = due today |
| WI | Working interest — operator's share of costs and revenue |
| ORRI | Overriding royalty interest — royalty carved out of WI |
| NRI | Net revenue interest — share of production revenue |
| JIB | Joint interest billing — billing between working interest partners |
| Suspense | Revenue held pending resolution (missing division orders, address, title issues) |

## Related Resources

- **Analytics repo:** `Formentera-Operations/analytics` — dbt models, staging SQL, gold model source
- **Brainstorm doc:** `docs/brainstorms/2026-02-26-oda-reports-claude-skills-brainstorm.md`
- **Source system:** Quorum ODA (Estuary CDC → ESTUARY_DB → dbt → FO_PRODUCTION_DB)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Formentera-Operations)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Formentera-Operations)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
