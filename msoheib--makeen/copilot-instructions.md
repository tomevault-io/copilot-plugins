## makeen

> - **Supabase Project ID**: `fbabpaorcvatejkrelrf`

# Real Estate Management Database Schema Documentation

## Project Information
- **Supabase Project ID**: `fbabpaorcvatejkrelrf`
- **Project Name**: `property-management` 
- **Region**: `eu-central-1`
- **Database Version**: PostgreSQL 15.8.1.093
- **Status**: ACTIVE_HEALTHY
- **Host**: `db.fbabpaorcvatejkrelrf.supabase.co`

## Schema Overview

The database is designed for a comprehensive real estate management system supporting property management, tenant/owner relationships, financial transactions, maintenance tracking, document management, and accounting functions.

### Core Business Entities
1. **People Management** - `profiles`, `clients`
2. **Property Management** - `properties`, `property_reservations`
3. **Contract Management** - `contracts` 
4. **Maintenance Operations** - `maintenance_requests`, `work_orders`
5. **Financial Management** - `vouchers`, `invoices`, `accounts`, `cost_centers`
6. **Communications** - `letters`, `issues`
7. **Document Management** - `documents`
8. **Asset Management** - `fixed_assets`

---

## Table Definitions

### 1. PROFILES
**Purpose**: Central table for all people in the system (users, tenants, owners, employees, contractors)

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `first_name` | text | YES | NULL | |
| `last_name` | text | YES | NULL | |
| `email` | text | YES | NULL | Unique |
| `role` | text | YES | `'tenant'` | CHECK: admin, manager, owner, tenant, buyer, employee, contractor |
| `phone` | text | YES | NULL | |
| `address` | text | YES | NULL | |
| `city` | text | YES | NULL | |
| `country` | text | YES | NULL | |
| `nationality` | text | YES | NULL | |
| `id_number` | text | YES | NULL | |
| `is_foreign` | boolean | YES | `false` | |
| `profile_type` | text | YES | NULL | CHECK: owner, tenant, buyer, employee, admin |
| `status` | text | YES | `'active'` | CHECK: active, inactive, suspended |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Referenced By
- `properties.owner_id` → `profiles.id`
- `contracts.tenant_id` → `profiles.id`
- `maintenance_requests.tenant_id` → `profiles.id`
- `work_orders.assigned_to` → `profiles.id`
- `vouchers.tenant_id` → `profiles.id`
- `vouchers.created_by` → `profiles.id`
- `invoices.tenant_id` → `profiles.id`
- `letters.sender_id` → `profiles.id`
- `issues.reported_by` → `profiles.id`
- `documents.uploaded_by` → `profiles.id`

---

### 2. PROPERTIES
**Purpose**: Property listings and management information

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `title` | text | NO | NULL | |
| `description` | text | YES | NULL | |
| `property_type` | text | NO | NULL | CHECK: apartment, villa, office, retail, warehouse |
| `status` | text | YES | `'available'` | CHECK: available, rented, maintenance, reserved |
| `address` | text | NO | NULL | |
| `city` | text | NO | NULL | |
| `country` | text | NO | NULL | |
| `neighborhood` | text | YES | NULL | |
| `area_sqm` | numeric | NO | NULL | |
| `bedrooms` | integer | YES | NULL | |
| `bathrooms` | integer | YES | NULL | |
| `price` | numeric | NO | NULL | |
| `payment_method` | text | YES | `'cash'` | CHECK: cash, installment |
| `owner_id` | uuid | YES | NULL | FK to profiles.id |
| `images` | text[] | YES | NULL | |
| `property_code` | text | YES | NULL | |
| `floor_number` | integer | YES | NULL | |
| `building_name` | text | YES | NULL | |
| `parking_spaces` | integer | YES | `0` | |
| `amenities` | text[] | YES | NULL | |
| `is_furnished` | boolean | YES | `false` | |
| `annual_rent` | numeric | YES | NULL | |
| `service_charge` | numeric | YES | `0` | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `owner_id` → `profiles.id` (ON DELETE RESTRICT)

#### Referenced By
- `contracts.property_id` → `properties.id`
- `maintenance_requests.property_id` → `properties.id`
- `vouchers.property_id` → `properties.id`
- `invoices.property_id` → `properties.id`
- `property_reservations.property_id` → `properties.id`
- `issues.property_id` → `properties.id`
- `fixed_assets.property_id` → `properties.id`

---

### 3. CONTRACTS
**Purpose**: Rental, sales, and management contracts

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `property_id` | uuid | YES | NULL | FK to properties.id |
| `tenant_id` | uuid | YES | NULL | FK to profiles.id |
| `start_date` | timestamptz | NO | NULL | |
| `end_date` | timestamptz | NO | NULL | |
| `rent_amount` | numeric | NO | NULL | |
| `payment_frequency` | text | YES | `'monthly'` | CHECK: monthly, quarterly, biannually, annually |
| `security_deposit` | numeric | NO | NULL | |
| `is_foreign_tenant` | boolean | YES | `false` | |
| `status` | text | YES | `'active'` | CHECK: active, expired, terminated, renewal |
| `documents` | text[] | YES | NULL | |
| `contract_number` | text | YES | NULL | |
| `contract_type` | text | YES | `'rental'` | CHECK: rental, sale, management |
| `auto_renewal` | boolean | YES | `false` | |
| `notice_period_days` | integer | YES | `30` | |
| `late_fee_percentage` | numeric | YES | `0` | |
| `utilities_included` | boolean | YES | `false` | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `property_id` → `properties.id` (ON DELETE RESTRICT)
- `tenant_id` → `profiles.id` (ON DELETE RESTRICT)

---

### 4. MAINTENANCE_REQUESTS
**Purpose**: Maintenance requests submitted by tenants or property managers

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `property_id` | uuid | YES | NULL | FK to properties.id |
| `tenant_id` | uuid | YES | NULL | FK to profiles.id |
| `title` | text | NO | NULL | |
| `description` | text | NO | NULL | |
| `status` | text | YES | `'pending'` | CHECK: pending, approved, in_progress, completed, cancelled |
| `priority` | text | YES | `'medium'` | CHECK: low, medium, high, urgent |
| `images` | text[] | YES | NULL | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `property_id` → `properties.id` (ON DELETE RESTRICT)
- `tenant_id` → `profiles.id` (ON DELETE RESTRICT)

#### Referenced By
- `work_orders.maintenance_request_id` → `maintenance_requests.id`

---

### 5. WORK_ORDERS
**Purpose**: Work orders assigned to contractors/employees for maintenance

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `maintenance_request_id` | uuid | YES | NULL | FK to maintenance_requests.id |
| `assigned_to` | uuid | YES | NULL | FK to profiles.id |
| `description` | text | NO | NULL | |
| `estimated_cost` | numeric | NO | NULL | |
| `actual_cost` | numeric | YES | NULL | |
| `start_date` | timestamptz | NO | NULL | |
| `completion_date` | timestamptz | YES | NULL | |
| `status` | text | YES | `'assigned'` | CHECK: assigned, in_progress, completed, cancelled |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `maintenance_request_id` → `maintenance_requests.id` (ON DELETE RESTRICT)
- `assigned_to` → `profiles.id` (ON DELETE RESTRICT)

---

### 6. VOUCHERS
**Purpose**: Financial vouchers (receipt, payment, journal entries)

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `voucher_type` | text | NO | NULL | CHECK: receipt, payment, journal |
| `voucher_number` | text | NO | NULL | Unique |
| `amount` | numeric | NO | NULL | |
| `currency` | text | YES | `'USD'` | |
| `status` | text | YES | `'draft'` | CHECK: draft, posted, cancelled |
| `description` | text | YES | NULL | |
| `property_id` | uuid | YES | NULL | FK to properties.id |
| `tenant_id` | uuid | YES | NULL | FK to profiles.id |
| `created_by` | uuid | YES | NULL | FK to profiles.id |
| `payment_method` | text | YES | NULL | CHECK: cash, bank_transfer, cheque, card |
| `cheque_number` | text | YES | NULL | |
| `bank_reference` | text | YES | NULL | |
| `account_id` | uuid | YES | NULL | FK to accounts.id |
| `cost_center_id` | uuid | YES | NULL | FK to cost_centers.id |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `property_id` → `properties.id` (ON DELETE RESTRICT)
- `tenant_id` → `profiles.id` (ON DELETE RESTRICT)
- `created_by` → `profiles.id` (ON DELETE RESTRICT)
- `account_id` → `accounts.id` (ON DELETE RESTRICT)
- `cost_center_id` → `cost_centers.id` (ON DELETE RESTRICT)

---

### 7. INVOICES
**Purpose**: VAT invoices and billing management

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `invoice_number` | text | NO | NULL | Unique |
| `property_id` | uuid | YES | NULL | FK to properties.id |
| `tenant_id` | uuid | YES | NULL | FK to profiles.id |
| `amount` | numeric | NO | NULL | |
| `vat_amount` | numeric | YES | `0` | |
| `total_amount` | numeric | NO | NULL | |
| `issue_date` | timestamptz | NO | NULL | |
| `due_date` | timestamptz | NO | NULL | |
| `status` | text | YES | `'draft'` | CHECK: draft, sent, paid, overdue, cancelled |
| `description` | text | YES | NULL | |
| `tax_rate` | numeric | YES | `0` | |
| `discount_amount` | numeric | YES | `0` | |
| `payment_terms` | text | YES | `'30 days'` | |
| `reference_number` | text | YES | NULL | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `property_id` → `properties.id` (ON DELETE RESTRICT)
- `tenant_id` → `profiles.id` (ON DELETE RESTRICT)

---

### 8. ACCOUNTS
**Purpose**: Chart of accounts for accounting system

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `account_code` | text | NO | NULL | Unique |
| `account_name` | text | NO | NULL | |
| `account_type` | text | NO | NULL | CHECK: asset, liability, equity, revenue, expense |
| `parent_account_id` | uuid | YES | NULL | FK to accounts.id (self-reference) |
| `is_active` | boolean | YES | `true` | |
| `description` | text | YES | NULL | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `parent_account_id` → `accounts.id` (self-referencing, ON DELETE RESTRICT)

#### Referenced By
- `vouchers.account_id` → `accounts.id`

#### Sample Data (Chart of Accounts)
```
1000 - ASSETS
  1100 - Current Assets
    1110 - Cash and Cash Equivalents
    1120 - Accounts Receivable
    1130 - Inventory
  1200 - Fixed Assets
    1210 - Property, Plant & Equipment
    1220 - Accumulated Depreciation

2000 - LIABILITIES
  2100 - Current Liabilities
    2110 - Accounts Payable
    2120 - Accrued Expenses
  2200 - Long-term Liabilities
    2210 - Long-term Debt

3000 - EQUITY
  3100 - Owner's Equity
    3110 - Capital
    3120 - Retained Earnings

4000 - REVENUE
  4100 - Rental Income
  4200 - Service Income

5000 - EXPENSES
  5100 - Operating Expenses
    5110 - Maintenance Expenses
    5120 - Administrative Expenses
```

---

### 9. COST_CENTERS
**Purpose**: Cost center management for expense allocation

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `code` | text | NO | NULL | Unique |
| `name` | text | NO | NULL | |
| `description` | text | YES | NULL | |
| `is_active` | boolean | YES | `true` | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Referenced By
- `vouchers.cost_center_id` → `cost_centers.id`

#### Sample Data
```
CC001 - Property Management
CC002 - Maintenance Operations  
CC003 - Administrative
CC004 - Marketing
CC005 - Legal and Compliance
```

---

### 10. CLIENTS
**Purpose**: External clients, suppliers, vendors, contractors

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `client_type` | text | NO | NULL | CHECK: buyer, supplier, vendor, contractor |
| `company_name` | text | YES | NULL | |
| `contact_person` | text | YES | NULL | |
| `email` | text | YES | NULL | Unique |
| `phone` | text | YES | NULL | |
| `address` | text | YES | NULL | |
| `city` | text | YES | NULL | |
| `country` | text | YES | NULL | |
| `tax_id` | text | YES | NULL | |
| `status` | text | YES | `'active'` | CHECK: active, inactive |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Referenced By
- `property_reservations.client_id` → `clients.id`

---

### 11. PROPERTY_RESERVATIONS
**Purpose**: Property booking and reservation system

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `property_id` | uuid | YES | NULL | FK to properties.id |
| `client_id` | uuid | YES | NULL | FK to clients.id |
| `reservation_date` | timestamptz | NO | NULL | |
| `expiry_date` | timestamptz | NO | NULL | |
| `deposit_amount` | numeric | NO | NULL | |
| `status` | text | YES | `'active'` | CHECK: active, expired, converted, cancelled |
| `notes` | text | YES | NULL | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `property_id` → `properties.id` (ON DELETE RESTRICT)
- `client_id` → `clients.id` (ON DELETE RESTRICT)

---

### 12. LETTERS
**Purpose**: Document and letter management system

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `recipient_type` | text | NO | NULL | CHECK: tenant, owner, client, supplier |
| `recipient_id` | uuid | YES | NULL | |
| `sender_id` | uuid | YES | NULL | FK to profiles.id |
| `subject` | text | NO | NULL | |
| `content` | text | NO | NULL | |
| `letter_type` | text | YES | NULL | CHECK: notice, reminder, legal, general |
| `status` | text | YES | `'draft'` | CHECK: draft, sent, delivered, read |
| `sent_date` | timestamptz | YES | NULL | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `sender_id` → `profiles.id` (ON DELETE RESTRICT)

---

### 13. ISSUES
**Purpose**: Issue tracking and complaint management

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `property_id` | uuid | YES | NULL | FK to properties.id |
| `reported_by` | uuid | YES | NULL | FK to profiles.id |
| `issue_type` | text | YES | NULL | CHECK: complaint, dispute, legal, administrative |
| `title` | text | NO | NULL | |
| `description` | text | NO | NULL | |
| `priority` | text | YES | `'medium'` | CHECK: low, medium, high, urgent |
| `status` | text | YES | `'open'` | CHECK: open, investigating, resolved, closed |
| `resolution` | text | YES | NULL | |
| `resolved_date` | timestamptz | YES | NULL | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `property_id` → `properties.id` (ON DELETE RESTRICT)
- `reported_by` → `profiles.id` (ON DELETE RESTRICT)

---

### 14. DOCUMENTS
**Purpose**: Document archive and file management

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `document_type` | text | NO | NULL | CHECK: contract, invoice, receipt, legal, insurance, maintenance, photo, other |
| `title` | text | NO | NULL | |
| `file_path` | text | NO | NULL | |
| `file_size` | bigint | YES | NULL | |
| `mime_type` | text | YES | NULL | |
| `related_entity_type` | text | YES | NULL | CHECK: property, tenant, contract, maintenance, work_order |
| `related_entity_id` | uuid | YES | NULL | |
| `uploaded_by` | uuid | YES | NULL | FK to profiles.id |
| `tags` | text[] | YES | NULL | |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `uploaded_by` → `profiles.id` (ON DELETE RESTRICT)

---

### 15. FIXED_ASSETS
**Purpose**: Fixed asset tracking and depreciation

#### Columns
| Column | Type | Nullable | Default | Constraints |
|--------|------|----------|---------|-------------|
| `id` | uuid | NO | `gen_random_uuid()` | Primary Key |
| `property_id` | uuid | YES | NULL | FK to properties.id |
| `asset_name` | text | NO | NULL | |
| `asset_type` | text | NO | NULL | |
| `purchase_date` | date | NO | NULL | |
| `purchase_price` | numeric | NO | NULL | |
| `depreciation_method` | text | YES | `'straight_line'` | CHECK: straight_line, declining_balance |
| `useful_life_years` | integer | NO | NULL | |
| `salvage_value` | numeric | YES | `0` | |
| `current_value` | numeric | YES | NULL | |
| `status` | text | YES | `'active'` | CHECK: active, disposed, sold |
| `created_at` | timestamptz | YES | `now()` | |
| `updated_at` | timestamptz | YES | `now()` | |

#### Foreign Keys
- `property_id` → `properties.id` (ON DELETE RESTRICT)

---

## Database Migrations

### Migration History
1. **20250610030118_add_missing_core_tables**: Initial creation of additional tables
2. **20250610030220_enhance_existing_tables_fixed**: Enhanced existing tables with additional fields
3. **20250610030308_add_default_chart_of_accounts**: Seeded default chart of accounts data

---

## PostgreSQL Extensions

### Active Extensions
- **pgcrypto** (v1.3): Cryptographic functions
- **uuid-ossp** (v1.1): UUID generation
- **pg_stat_statements** (v1.10): Query performance tracking
- **pg_graphql** (v1.5.11): GraphQL support
- **supabase_vault** (v0.3.1): Vault extension
- **plpgsql** (v1.0): PL/pgSQL procedural language

### Available Extensions (Not Installed)
Available but not currently activated: postgis, vector, pgjwt, http, wrappers, pg_net, timescaledb, pg_cron, and many others for advanced functionality.

---

## Entity Relationship Summary

### Core Relationships
```
profiles (1) ←→ (M) properties [owner_id]
properties (1) ←→ (M) contracts [property_id]
profiles (1) ←→ (M) contracts [tenant_id]
properties (1) ←→ (M) maintenance_requests [property_id]
maintenance_requests (1) ←→ (M) work_orders [maintenance_request_id]
profiles (1) ←→ (M) work_orders [assigned_to]
properties (1) ←→ (M) vouchers [property_id]
profiles (1) ←→ (M) vouchers [tenant_id, created_by]
accounts (1) ←→ (M) vouchers [account_id]
cost_centers (1) ←→ (M) vouchers [cost_center_id]
properties (1) ←→ (M) invoices [property_id]
profiles (1) ←→ (M) invoices [tenant_id]
clients (1) ←→ (M) property_reservations [client_id]
properties (1) ←→ (M) property_reservations [property_id]
profiles (1) ←→ (M) letters [sender_id]
properties (1) ←→ (M) issues [property_id]
profiles (1) ←→ (M) issues [reported_by]
profiles (1) ←→ (M) documents [uploaded_by]
properties (1) ←→ (M) fixed_assets [property_id]
accounts (1) ←→ (M) accounts [parent_account_id] (self-referencing)
```

### Business Logic Constraints
- Properties can have multiple contracts over time but typically one active contract
- Maintenance requests can generate multiple work orders
- Financial vouchers must be linked to valid accounts from chart of accounts
- All monetary transactions require proper account code classification
- Document management supports tagging and categorization
- Fixed assets track depreciation and current values
- Cost centers enable expense allocation across business units

---

## Database Statistics
- **Total Tables**: 15
- **Total Columns**: 247
- **Total Foreign Key Relationships**: 25
- **Total Check Constraints**: 48
- **Total Unique Constraints**: 6

---

## Performance Considerations
- All tables use UUID primary keys for distributed system compatibility
- Automatic timestamps (created_at, updated_at) on all entities
- Foreign key constraints ensure referential integrity
- Check constraints enforce business rules at database level
- Text arrays used for flexible multi-value fields (images, amenities, tags)
- Hierarchical account structure supports complex financial reporting

---

## Data Security
- Row Level Security (RLS) is currently disabled on all tables
- Supabase authentication system integration available
- PostgreSQL role-based security can be implemented
- Audit trail maintained through timestamp fields
- Foreign key constraints prevent orphaned records

---

*Last Updated: December 2024*  
*Database Schema Version: 3 migrations applied*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/msoheib) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
