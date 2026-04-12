## zarabooks

> > This file maintains context for Claude Code sessions. Updated as features are added.

# Zara Books - Project Memory

> This file maintains context for Claude Code sessions. Updated as features are added.

---

## Quick Reference

| Item | Value |
|------|-------|
| **Project** | Zara Books - Accounting Application |
| **Target Users** | CA firms, Startups (India) |
| **Deployed URL** | https://scintillating-stillness-production-02d4.up.railway.app |
| **Repository** | https://github.com/kbr1991/ZaraBooks |
| **Current Version** | 1.4.1 |

### Tech Stack
- **Frontend:** React 18, TypeScript, Tailwind CSS, React Query, React Router
- **Backend:** Express.js, TypeScript
- **Database:** PostgreSQL with Drizzle ORM (Railway)
- **UI:** Radix UI (shadcn/ui), Lucide icons
- **Testing:** Vitest (123 tests: 74 unit + 49 integration)

### Default Login (Super Admin)
- Email: `admin@zarabooks.com`
- Password: `Admin@123`

### New Portals
- `/admin` - Super admin dashboard (requires super_admin role)
- `/partner` - Partner portal (requires partner access)
- `/partner/register` - Partner registration (public)

---

## Project Structure

```
ZaraBooks/
├── client/src/
│   ├── components/
│   │   ├── ui/              # shadcn/ui components
│   │   ├── layout/          # Sidebar, MainLayout
│   │   └── accounting/      # Domain components
│   ├── pages/               # Route pages (40+ pages)
│   ├── hooks/               # Custom hooks
│   └── lib/                 # Utilities
├── server/src/
│   ├── routes/              # API endpoints
│   ├── services/            # Business logic
│   ├── middleware/          # Auth middleware
│   └── db/                  # Database config
├── shared/
│   └── schema.ts            # Drizzle schema (28+ tables)
├── tests/
│   ├── unit/                # Unit tests
│   └── integration/         # DB integration tests
├── CLAUDE.md                # This file
├── DEVELOPMENT_SOP.md       # Development standards
├── CHANGELOG.md             # Version history
└── README.md                # Project overview
```

---

## Completed Features

### Core Accounting
| Module | Description | Status |
|--------|-------------|--------|
| Multi-Company | Multiple companies per user | Done |
| Chart of Accounts | 5-level hierarchy, India GAAP template | Done |
| Journal Entries | Double-entry with validation | Done |
| Trial Balance | Real-time calculation, drill-down | Done |
| Balance Sheet | Schedule III format | Done |
| Profit & Loss | Schedule III format | Done |
| Cash Flow | Indirect method | Done |
| Ledger View | Account transaction history | Done |

### Sales Module
| Module | Description | Status |
|--------|-------------|--------|
| Customers | Customer management with receivables | Done |
| Quotes | Estimates with convert-to-invoice | Done |
| Sales Orders | Order tracking | Done |
| Invoices | GST-compliant invoicing | Done |
| Payments Received | Customer payment tracking | Done |
| Credit Notes | Customer refunds | Done |

### Purchases Module
| Module | Description | Status |
|--------|-------------|--------|
| Vendors | Vendor management with payables | Done |
| Purchase Orders | PO management | Done |
| Bills | Vendor invoice tracking | Done |
| Expenses | Expense with approval workflow | Done |
| Payments Made | Vendor payment tracking | Done |
| Debit Notes | Vendor adjustments | Done |

### Banking
| Module | Description | Status |
|--------|-------------|--------|
| Bank Accounts | Multi-account management | Done |
| Bank Reconciliation | Transaction matching | Done |
| Bank Import | CSV/OFX import | Done |

### Inventory
| Module | Description | Status |
|--------|-------------|--------|
| Products & Services | Item catalog with HSN/SAC | Done |

### Compliance (India)
| Module | Description | Status |
|--------|-------------|--------|
| GST Config | Multi-GSTIN support | Done |
| GSTR-1 | Outward supplies data | Done |
| GSTR-3B | Summary return | Done |
| ITC Register | Input tax credit tracking | Done |
| TDS Register | TDS deduction tracking | Done |
| TDS Challans | Payment tracking | Done |

### Administration
| Module | Description | Status |
|--------|-------------|--------|
| User Management | Invite users with roles | Done |
| Audit Log | Activity tracking | Done |
| Settings | Company, fiscal year config | Done |

---

## Pending Features

### Phase 2 - Compliance Enhancement
- [ ] E-Invoice generation (NIC API)
- [ ] E-Way Bill generation
- [ ] GSTR-2A/2B reconciliation
- [ ] Form 26AS download (TRACES)
- [ ] TDS return preparation

### Phase 3 - Integration
- [ ] Practice Manager sync
- [ ] Bank statement auto-import
- [ ] WhatsApp notifications
- [ ] Email reminders

### Phase 4 - AI & Analytics
- [ ] AI Assistant (Claude API)
- [ ] Custom dashboard builder
- [ ] Advanced analytics
- [ ] Predictive insights

---

## Database Schema Overview

### Core Tables (15)
- `users`, `companies`, `companyUsers`, `userInvitations`
- `fiscalYears`, `chartOfAccounts`, `costCenters`
- `journalEntries`, `journalEntryLines`
- `parties`, `bankAccounts`, `attachments`, `auditLog`
- `recurringEntryTemplates`, `trialBalanceCache`

### Transaction Tables (6)
- `invoices`, `invoiceLines`
- `expenses`
- `quotes`, `salesOrders`, `purchaseOrders`
- `bills`, `creditNotes`, `debitNotes`
- `paymentsReceived`, `paymentsMade`

### GST Tables (6)
- `gstConfig`, `gstr1Entries`, `gstr3bSummary`
- `itcRegister`, `gstPayments`, `hsnSacMaster`

### TDS Tables (4)
- `tdsSections`, `tdsDeductions`, `tdsChallans`, `form26asEntries`

### Integration Tables (4)
- `pmIntegrationConfig`, `pmSyncLog`
- `aiConversations`, `financialStatementRuns`

---

## API Routes

### Authentication
```
POST /api/auth/register
POST /api/auth/login
GET  /api/auth/me
POST /api/auth/logout
```

### Companies
```
GET    /api/companies
POST   /api/companies
POST   /api/companies/:id/select
```

### Chart of Accounts
```
GET    /api/chart-of-accounts
POST   /api/chart-of-accounts
GET    /api/chart-of-accounts/ledgers/list
GET    /api/chart-of-accounts/:id/transactions
```

### Journal Entries
```
GET    /api/journal-entries
POST   /api/journal-entries
PATCH  /api/journal-entries/:id
POST   /api/journal-entries/:id/post
POST   /api/journal-entries/:id/reverse
```

### Sales
```
/api/customers, /api/quotes, /api/sales-orders
/api/invoices, /api/payments-received, /api/credit-notes
```

### Purchases
```
/api/vendors, /api/purchase-orders, /api/bills
/api/expenses, /api/payments-made, /api/debit-notes
```

### Banking
```
/api/bank-accounts, /api/bank-reconciliation, /api/bank-import
```

### Reports
```
GET /api/trial-balance
GET /api/financial-statements/balance-sheet
GET /api/financial-statements/profit-loss
GET /api/financial-statements/cash-flow
```

### Compliance
```
/api/gst/*, /api/tds/*
```

---

## Key Files Reference

| Purpose | File |
|---------|------|
| Database Schema | `shared/schema.ts` |
| API Routes | `server/src/routes/*.ts` |
| Pages | `client/src/pages/*.tsx` |
| Sidebar | `client/src/components/layout/Sidebar.tsx` |
| App Routes | `client/src/App.tsx` |
| Test Setup | `tests/setup.ts` |
| Seed Data | `server/src/db/seed.ts` |

---

## Change Log

### 2025-02-09 (Session 6 - Multi-Standard Templates & Entity-Type Awareness)

**New Features:**

1. **Entity-Type-Aware India GAAP Templates:**
   - Company (Pvt Ltd/Public/OPC): Share Capital, Reserves & Surplus
   - Partnership: Partners' Capital, Partners' Current Accounts
   - LLP: Partners' Contribution, Reserves
   - Proprietorship: Proprietor's Capital, Drawings Account

2. **Multiple Accounting Standards:**
   - India GAAP (auto-detect based on company type)
   - US GAAP (Common Stock, Retained Earnings, AOCI)
   - IFRS (Issued Capital, Revaluation Surplus, Non-controlling Interests)

3. **Template Selection Dialog:**
   - Visual selection of accounting standard
   - Auto-detect option reads company type and applies correct template
   - Info panel explains equity structure differences

4. **Backend Enhancements:**
   - `getTemplateForCompanyType()` helper function
   - `autoDetect` parameter in template application
   - Extended `gaap_standard` enum with Partnership/LLP/Proprietorship variants

**Files Modified:**
- `server/src/db/seed.ts` - Complete rewrite with 6 templates, modular account structures
- `server/src/routes/chartOfAccounts.ts` - Auto-detect logic, company type mapping
- `shared/schema.ts` - Extended gaapStandardEnum
- `client/src/pages/ChartOfAccounts.tsx` - Template selection dialog

---

### 2025-02-09 (Session 5 - Chart of Accounts CRUD Fix)

**Critical Fix: Add Account Button Was Non-Functional**
- The "Add Account" button existed but had no onClick handler
- No form dialog was implemented for creating accounts
- Backend endpoints existed but frontend was incomplete

**Changes Made:**

1. **Full CRUD Implementation for Chart of Accounts:**
   - Added `showAddDialog`, `showEditDialog`, `showDeleteDialog` state
   - Added `formData` state for form fields
   - Added `createAccountMutation` for POST /api/chart-of-accounts
   - Added `updateAccountMutation` for PATCH /api/chart-of-accounts/:id
   - Added `deleteAccountMutation` for DELETE /api/chart-of-accounts/:id
   - Added `initializeTemplateMutation` for POST /api/chart-of-accounts/templates/apply

2. **Add Account Dialog with Fields:**
   - Account Code, Account Name (required)
   - Account Type (asset/liability/equity/income/expense)
   - Parent Account (dropdown of group accounts)
   - Is Group Account checkbox
   - Description
   - Opening Balance with Debit/Credit type
   - GST Settings (applicable, rate, HSN/SAC code)

3. **Edit Account Dialog:**
   - Same fields as Add dialog
   - Fetches full account details on edit

4. **Delete Confirmation Dialog**

5. **Template Initialization:**
   - Template selection dialog with multiple accounting standards
   - New backend endpoint: POST /api/chart-of-accounts/templates/apply
   - Applies comprehensive templates (90+ accounts each)

6. **AccountRow Edit/Delete Buttons:**
   - Edit button for all accounts
   - Delete button for non-group accounts

**Files Modified:**
- `client/src/pages/ChartOfAccounts.tsx` - Complete CRUD implementation
- `server/src/routes/chartOfAccounts.ts` - Added /templates/apply endpoint

---

### 2025-02-09 (Session 4 - CRUD Delete & Template Fixes)

**Delete Functionality Added:**
- Added delete mutations and buttons to: Invoices, SalesOrders, Bills, PurchaseOrders, CreditNotes, DebitNotes, Expenses, PaymentsReceived, PaymentsMade
- Each page now has proper delete with confirmation dialogs
- Backend status checks enforced (e.g., only draft invoices can be deleted)

**Document Template Fixes:**
- Fixed invoice/quote line item amount calculation (qty × rate, not including tax)
- Fixed HSN/SAC field mapping (`hsnSacCode` → `hsnSac`)
- Removed status badge for non-overdue documents (no more "DRAFT" on final invoices)
- Removed "Generated by Zara Books" footer
- Fixed double commas in addresses with `cleanAddressPart()` helper
- Improved print CSS with proper `@page` rules

**Bug Fixes:**
- Purchase Orders: Fixed field name mismatch (`poNumber`/`poDate` → `orderNumber`/`orderDate`)
- Bills: Fixed delete to allow 'draft' and 'pending' (open) statuses
- Bills: Added journal entry cleanup on deletion

**Files Modified:**
- `client/src/pages/Invoices.tsx` - Added deleteInvoiceMutation
- `client/src/pages/SalesOrders.tsx` - Added deleteOrderMutation
- `client/src/pages/Bills.tsx` - Added deleteBillMutation, fixed status condition
- `client/src/pages/PurchaseOrders.tsx` - Fixed interface fields, added deletePOMutation
- `client/src/pages/CreditNotes.tsx` - Added deleteCreditNoteMutation
- `client/src/pages/DebitNotes.tsx` - Added deleteDebitNoteMutation
- `client/src/pages/Expenses.tsx` - Added deleteExpenseMutation
- `client/src/pages/PaymentsReceived.tsx` - Added deletePaymentMutation
- `client/src/pages/PaymentsMade.tsx` - Added deletePaymentMutation
- `client/src/pages/Quotes.tsx` - Already had delete (verified)
- `client/src/lib/document-templates/templates/*.ts` - All 4 templates updated
- `server/src/routes/bills.ts` - Fixed delete status check, added journal cleanup

### 2025-02-07 (Session 2 - Zoho Books Parity)

**New Pages Created (11):**
- `Quotes.tsx` - Quote management with convert-to-invoice
- `SalesOrders.tsx` - Sales order tracking
- `PaymentsReceived.tsx` - Customer payment tracking
- `CreditNotes.tsx` - Customer credit notes
- `Bills.tsx` - Vendor bill management
- `PurchaseOrders.tsx` - Purchase order management
- `PaymentsMade.tsx` - Vendor payment tracking
- `DebitNotes.tsx` - Vendor debit notes
- `BankAccounts.tsx` - Bank account management
- `BankReconciliation.tsx` - Transaction reconciliation
- `Products.tsx` - Products & services catalog

**Sidebar Reorganization:**
- Items: Products & Services
- Sales: Customers, Quotes, Sales Orders, Invoices, Payments Received, Credit Notes
- Purchases: Vendors, Purchase Orders, Bills, Payments Made, Expenses, Debit Notes
- Banking: Bank Accounts, Bank Reconciliation, Bank Import
- Accountant: Chart of Accounts, Journal Entries, Ledger View, Recurring Entries, Parties
- Reports: Trial Balance, Balance Sheet, P&L, Cash Flow, Aging Reports
- Compliance: GST Returns, TDS Register

**Files Modified:**
- `client/src/App.tsx` - Added routes for 11 new pages
- `client/src/components/layout/Sidebar.tsx` - Complete restructure

### 2025-02-07 (Session 1 - Test Fixes)

**Test Infrastructure:**
- Fixed all 123 tests (74 unit + 49 integration)
- Fixed column name `passwordHash` → `password` in test files
- Fixed decimal assertions (`'18.00'` format)
- Added unique constraint on `companies.pan`
- Added unique index on `(companyId, code)` for chart_of_accounts
- Updated test database URL

**Files Modified:**
- `shared/schema.ts` - Added uniqueIndex import, constraints
- `tests/setup.ts` - Fixed default DB URL
- `tests/integration/*.ts` - Fixed column names and assertions

### 2025-02-06 (Initial Development)

**Core Features Implemented:**
- Multi-company support with user roles
- Chart of Accounts with India GAAP template
- Journal Entry system with double-entry validation
- Trial Balance with real-time calculation
- Financial Statements (Schedule III)
- GST module (GSTR-1, GSTR-3B, ITC)
- TDS module
- User management with invitations
- Customer and Vendor management
- Invoice and Expense tracking

---

## Environment Variables

```env
# Database
DATABASE_URL=postgresql://localhost/zarabooks

# Session
SESSION_SECRET=your-session-secret

# Email (Resend - required for user invitations)
RESEND_API_KEY=re_xxxxxxxxxxxxx
EMAIL_FROM=Zara Books <noreply@yourdomain.com>
APP_URL=https://your-app-url.com

# AI (optional)
ANTHROPIC_API_KEY=your-api-key

# E-Invoice (optional)
NIC_EINVOICE_CLIENT_ID=
NIC_EINVOICE_CLIENT_SECRET=

# TRACES (optional)
TRACES_API_TAN=
TRACES_API_USER=
TRACES_API_PASSWORD=
```

---

## Deployment

### Railway
```bash
# Deploy
npx @railway/cli up

# Check status
npx @railway/cli status

# View logs
npx @railway/cli logs

# Health check
curl https://scintillating-stillness-production-02d4.up.railway.app/api/health
```

### Database
```bash
# Push schema
npm run db:push

# Seed data
npm run db:seed

# Open Drizzle Studio
npm run db:studio
```

---

## Session Notes

_Add notes during development sessions_

### 2026-02-09 (Session 8 - Email Service & User Management Fixes)

**User Management Fixes:**
- Fixed API endpoints in UserManagement.tsx (`/api/users/*` instead of `/api/companies/*`)
- Fixed JSON parsing error when inviting users
- Added missing `is_active` column to `user_invitations` table in production database

**Email Service Setup (Resend):**
- Integrated Resend for transactional emails
- Created email service with HTML/text templates (`server/src/services/email.ts`)
- Professional branded invitation emails
- Welcome emails after accepting invitations
- Password reset email templates (ready for future use)
- Created `/accept-invite` page for handling invitation links
- Added redirect parameter support to Login/Register pages

**Files Added:**
- `server/src/services/email.ts` - Email service with Resend integration and templates
- `client/src/pages/AcceptInvite.tsx` - Invitation acceptance page

**Files Modified:**
- `server/src/routes/userManagement.ts` - Added email sending on invite/accept
- `client/src/App.tsx` - Added /accept-invite route
- `client/src/pages/Login.tsx` - Added redirect parameter support
- `client/src/pages/Register.tsx` - Added redirect parameter support
- `client/src/pages/UserManagement.tsx` - Fixed API endpoints

**Environment Variables (Railway):**
- `RESEND_API_KEY` - Resend API key for email sending
- `EMAIL_FROM` - Sender address (using `onboarding@resend.dev` until domain verified)
- `APP_URL` - Application URL for email links

**Database Changes:**
- Added `is_active` column to `user_invitations` table

**Pending:**
- Verify `zarabooks.app` domain in Resend (DNS propagation)
- Once verified, update `EMAIL_FROM` to use custom domain

---

### 2026-02-10 (Session 9 - Multi-Tenancy & Partner Program)

**Major Feature: Multi-Tenancy with Partner/Reseller Program**

New hierarchy: Partner (optional) → Tenant → Company → Data

**Database Schema (8 new tables):**
- `partners` - CA firms/resellers with tier system (bronze/silver/gold/platinum)
- `tenants` - Subscribing organizations (billable entities)
- `partner_users` - Partner staff with roles
- `tenant_users` - Tenant access control
- `subscription_plans` - Plan definitions (free/starter/professional/enterprise)
- `subscriptions` - Subscription records
- `commissions` - Partner earnings tracking
- `partner_payouts` - Payout processing

**Backend Routes:**
- `/api/admin/*` - Platform management (dashboard, tenants, partners, subscriptions, commissions, payouts)
- `/api/partner/*` - Partner portal (dashboard, clients, commissions, payouts, team)
- `/api/tenant/*` - Tenant management (dashboard, companies, users, subscription)

**Frontend Pages:**
- Admin portal: Dashboard, Tenants, Partners, Subscriptions, Commissions, Payouts
- Partner portal: Dashboard, Clients, Commissions, Payouts, Team, Register
- Layouts: AdminLayout.tsx, PartnerLayout.tsx

**Auth Enhancements:**
- Multi-context login (super_admin, partner, tenant)
- New middleware: requireTenant, requirePartner, requireSuperAdmin
- Context switching between partner and tenant modes

**Files Added (38 files):**
- `server/src/routes/admin/*.ts` (7 files)
- `server/src/routes/partner/*.ts` (7 files)
- `server/src/routes/tenant/*.ts` (5 files)
- `client/src/pages/admin/*.tsx` (6 files)
- `client/src/pages/partner/*.tsx` (6 files)
- `client/src/components/layout/AdminLayout.tsx`
- `client/src/components/layout/PartnerLayout.tsx`

**Files Modified:**
- `shared/schema.ts` - Added 8 new tables, relations, and tenantId to companies
- `server/src/middleware/auth.ts` - Extended session, new middleware
- `server/src/routes/auth.ts` - Multi-context login flow
- `server/src/index.ts` - Registered new routes
- `client/src/App.tsx` - Added admin/partner routes with guards

**Deployment:**
- Pushed to GitHub and Railway
- Database schema updated

**Pending:**
- Create test user with correct email for login
- Seed initial subscription plans
- Test partner registration flow

---

### 2026-02-11 (Session 10 - Document Lifecycle: Download, Edit, Delete, Cancel)

**Major Fix: Full Document Lifecycle for All Document Types**

Previously, PurchaseOrders, Bills, CreditNotes, DebitNotes became unusable after being issued/sent - no download, no edit, no cancel. This session fixed all of it.

**Phase 1 - Template System Extended (4 new document types):**
- Added `purchase_order`, `bill`, `credit_note`, `debit_note` to `DocumentType` union
- Added `getPartyLabel()` helper - returns "Vendor" for PO/bill/debit_note, "Bill To" for others
- All 4 templates (classic, modern, professional, minimal) now use dynamic party labels
- `TemplateSelector` component widened to accept all 7 document types

**Phase 2 - Download/Print Added to 4 Pages:**
- PurchaseOrders, Bills, CreditNotes, DebitNotes now have download/print dropdown in table row actions
- Works for ALL statuses (not just draft)
- Dropdown: Quick Download, Print, Choose Template
- Uses TemplateSelector for template selection
- Each page builds proper `DocumentData` with correct type, vendor/customer mapping, and date fields

**Phase 3 - Edit Buttons Fixed on 3 Pages:**
- PurchaseOrders: Edit works for draft + issued statuses, fetches full PO with lines, populates form
- Bills: Edit works for draft + open statuses, fetches full bill with lines, populates form
- SalesOrders: Edit works for open + confirmed statuses, fetches full SO with lines, populates form
- All 3 use `updateMutation` with PATCH, `useEffect` to reset form on dialog close, conditional dialog title/footer

**Phase 4 - Backend Restrictions Relaxed + Cancel Endpoints:**
- `purchaseOrders.ts`: PATCH allows draft+issued, DELETE allows draft+issued (guards converted-to-bill), added `POST /:id/cancel`
- `salesOrders.ts`: PATCH allows draft+confirmed, DELETE allows draft+confirmed (guards converted-to-invoice), added `POST /:id/cancel`
- `invoices.ts`: Added `POST /:id/cancel` (sent only, no payments, reverses journal entries)
- `bills.ts`: Added `POST /:id/cancel` (pending/open only, no payments, reverses journal entries)
- `creditNotes.ts`: Added `POST /:id/cancel` (issued only, not applied, reverses journal entries)
- `debitNotes.ts`: Added `POST /:id/cancel` (issued only, not applied, reverses journal entries)

**Phase 5 - Cancel Buttons on All 6 Frontend Pages:**
- PurchaseOrders: Cancel for issued/acknowledged (not converted to bill)
- SalesOrders: Cancel for confirmed/processing (not converted to invoice)
- Invoices: Cancel for sent (no payments, warns about journal reversal)
- Bills: Cancel for pending/open (no payments, warns about journal reversal)
- CreditNotes: Cancel for issued (not applied)
- DebitNotes: Cancel for issued (not applied)

**Files Modified (19 files, +1334/-94 lines):**
- `client/src/components/document/TemplateSelector.tsx` - Widened documentType prop
- `client/src/lib/document-templates/types.ts` - Extended DocumentType union
- `client/src/lib/document-templates/index.ts` - Added getPartyLabel, extended getDocumentTitle/getDateLabel
- `client/src/lib/document-templates/templates/classic.ts` - Dynamic party labels
- `client/src/lib/document-templates/templates/modern.ts` - Dynamic party labels
- `client/src/lib/document-templates/templates/professional.ts` - Dynamic party labels
- `client/src/lib/document-templates/templates/minimal.ts` - Dynamic party labels
- `client/src/pages/PurchaseOrders.tsx` - Download, edit, cancel
- `client/src/pages/Bills.tsx` - Download, edit, cancel
- `client/src/pages/CreditNotes.tsx` - Download, cancel
- `client/src/pages/DebitNotes.tsx` - Download, cancel
- `client/src/pages/Invoices.tsx` - Cancel
- `client/src/pages/SalesOrders.tsx` - Edit, cancel
- `server/src/routes/purchaseOrders.ts` - Relaxed PATCH/DELETE, cancel endpoint
- `server/src/routes/salesOrders.ts` - Relaxed PATCH/DELETE, cancel endpoint
- `server/src/routes/invoices.ts` - Cancel endpoint with journal reversal
- `server/src/routes/bills.ts` - Cancel endpoint with journal reversal
- `server/src/routes/creditNotes.ts` - Cancel endpoint with journal reversal
- `server/src/routes/debitNotes.ts` - Cancel endpoint with journal reversal

**Deployment:**
- Pushed to GitHub (`bff6d08`)
- Deployed to Railway via `railway up`

**Document Lifecycle Status After This Session:**

| Page | Download/Print | Edit | Delete | Cancel |
|------|---------------|------|--------|--------|
| PurchaseOrders | All statuses | Draft + Issued | Draft + Issued | Issued/Acknowledged |
| Bills | All statuses | Draft + Open | Draft + Open | Pending/Open (no payments) |
| CreditNotes | All statuses | - | Draft | Issued (not applied) |
| DebitNotes | All statuses | - | Draft | Issued (not applied) |
| SalesOrders | All statuses | Open + Confirmed | Draft + Confirmed | Confirmed/Processing |
| Invoices | All statuses | Draft | Draft | Sent (no payments) |
| Quotes | All statuses | Draft + Sent | Draft + Sent | N/A |

---

### 2026-02-26 (Session 11 - Debug & Bug Fixes)

**Root Cause Analysis & Fixes:**

1. **Test Database Schema Out of Sync (43 integration tests failing):**
   - All integration tests were failing with `column "tenant_id" of relation "companies" does not exist`
   - Root cause: Session 9 added `tenant_id` to the `companies` table in `shared/schema.ts` but the test database (`zarabooks_test`) was never updated
   - Fix: Pushed schema to test database — `DATABASE_URL=postgresql://localhost/zarabooks_test npx drizzle-kit push --force`
   - All 151 tests now passing (102 unit + 49 integration)

2. **Partner Registration 404 Error:**
   - `client/src/pages/partner/Register.tsx` was calling `/api/partner/register`
   - Actual backend endpoint is `/api/partner/auth/register` (mounted under `/auth` subrouter in `server/src/routes/partner/index.ts`)
   - Fix: Corrected the fetch URL in `Register.tsx`

3. **Chart of Accounts: Unsafe Account Deletion:**
   - `server/src/routes/chartOfAccounts.ts` had a `TODO` — deletion never checked if an account had existing journal entry lines
   - Could create orphaned journal entry references
   - Fix: Added `journalEntryLines` lookup before deletion; returns 400 if any exist

4. **Bank Reconciliation: Unreconciled Endpoint Returned All Transactions:**
   - `server/src/routes/bankReconciliation.ts` had a `TODO` — always returned `isReconciled: false` without filtering
   - Fix: Queries `bankReconciliationLines` for `isReconciled = true` entries and excludes them from results

**Files Modified (3 files, +31/-15 lines):**
- `client/src/pages/partner/Register.tsx` - Fixed API endpoint path
- `server/src/routes/chartOfAccounts.ts` - Added journal entry safety check before deletion
- `server/src/routes/bankReconciliation.ts` - Filter reconciled entries from unreconciled endpoint

**Deployment:**
- Pushed to GitHub (`2742d28`)
- Auto-deployed to Railway via GitHub integration
- Health check confirmed: `{"status":"ok","service":"zara-books"}`

**Note on Test DB Maintenance:**
After any schema change to `shared/schema.ts`, run the following to keep the test database in sync:
```bash
DATABASE_URL=postgresql://localhost/zarabooks_test npx drizzle-kit push --force
```

---

*Last Updated: 2026-02-26*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KBR-and-Co)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/KBR-and-Co)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
