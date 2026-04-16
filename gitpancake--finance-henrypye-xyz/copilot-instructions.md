## finance-henrypye-xyz

> Personal finance dashboard. Next.js 16 + TypeScript + Tailwind CSS 4 + Supabase.

# finance.henrypye.xyz

Personal finance dashboard. Next.js 16 + TypeScript + Tailwind CSS 4 + Supabase.

## Commands

```bash
npm run dev          # Start dev server (localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
```

## Architecture

### Auth Flow

1. User authenticates via Firebase Authentication (email/password)
2. Firebase UID is linked to a `finance_users` profile in Supabase
3. Legacy path: `POST /api/auth` with username/password, bcrypt comparison against `finance_users`
4. On first-ever login (0 users), auto-creates admin from `AUTH_USERNAME`/`AUTH_PASSWORD` env vars and backfills existing data
5. Cookie `finance-auth` set to `user:<uuid>` (httpOnly, 30-day TTL)
6. `GET /api/auth` reads cookie, looks up user, returns `{ authenticated, userId, username, isAdmin }`
7. `DELETE /api/auth` clears cookie (logout)

Key files: `lib/auth.ts` (session helpers), `lib/firebase.ts` (client Firebase), `lib/firebase-admin.ts` (server Firebase Admin), `lib/firebase-users.ts` (Firebase user management), `app/api/auth/route.ts` (login/logout/check), `contexts/AuthContext.tsx` (client-side context), `components/AuthGate.tsx` (gate + provider nesting)

### Provider Nesting

`AuthGate` handles provider nesting when authenticated:

```
AuthGate (login gate)
  └─ AuthProvider (user info + logout)
       └─ FinanceProvider (userId prop → scoped data)
            └─ CurrencyProvider (exchange rates)
                 └─ Shell (sidebar nav)
                      └─ {children}
```

`layout.tsx` just renders `<AuthGate>{children}</AuthGate>`.

### Data Flow

All data uses optimistic updates via React Context:

1. Component dispatches an action (e.g. `ADD_DEBT`)
2. `FinanceContext` reducer updates state immediately
3. `persistAction(action, userId)` writes to Supabase in the background
4. On mount, `fetchAllData(userId)` loads everything from Supabase scoped to the user

All Supabase queries include `.eq("user_id", userId)` for data isolation.

### Key Directories

```
app/                  # Next.js App Router pages
  [section]/page.tsx  # Each tab is a client component page
  shared/page.tsx     # Shared expense categories
  settings/page.tsx   # Wallet address management
  api/
    auth/route.ts     # Login, session check, logout
    users/route.ts    # User management
    report/route.ts   # AI report generation (auth-protected)
    nfts/             # NFT data from OpenSea
    wallets/balances/ # On-chain wallet balance lookups
    shared/extract/   # Receipt image date extraction (AI)
components/           # Reusable UI components
  EditableTable.tsx   # Generic CRUD table (used by most pages)
  Shell.tsx           # Sidebar nav + user info + logout
  AuthGate.tsx        # Auth gate + provider nesting
  SummaryCard.tsx     # Dashboard summary cards
  CurrencyToggle.tsx  # Currency selector
  CurrencyBadge.tsx   # Currency display badge
  NetWorthBar.tsx     # Dashboard net worth visualization
  ReceiptUpload.tsx   # Receipt image upload with AI date extraction
  profile-dialog.tsx  # User profile editing dialog
  ui/                 # shadcn/ui primitives
  animate-ui/         # Animated UI components
contexts/
  AuthContext.tsx      # User info + logout (useAuth hook)
  FinanceContext.tsx   # Main state (reducer + Supabase persistence, userId-scoped)
  CurrencyContext.tsx  # Exchange rates + conversion helpers
  SharedContext.tsx    # Shared expense categories + items
lib/
  auth.ts             # Server-side session helpers (getSession, setSession, clearSession)
  firebase.ts         # Client-side Firebase init
  firebase-admin.ts   # Server-side Firebase Admin SDK
  firebase-users.ts   # Firebase user management helpers
  types.ts            # All TypeScript interfaces
  constants.ts        # Currencies, tax brackets, nav items, default state
  supabase.ts         # DB client, mapping helpers, CRUD functions (all userId-scoped)
  shared-supabase.ts  # Shared category/item Supabase operations
  opensea.ts          # OpenSea NFT API integration
  storage.ts          # Supabase Storage helpers (avatars, receipts)
  subscriptions.ts    # Subscription-related helpers
  format.ts           # formatMoney, formatMoneyShort, formatCrypto
  tax.ts              # Canadian tax calculation logic
  currency.ts         # Exchange rate fetching
  utils.ts            # General utilities (cn helper)
  get-strict-context.tsx # Strict context hook factory
  actions/            # Server actions (profile updates)
migrations/           # SQL migration files (run in order, 001-019)
```

### Adding a New Data Section

Follow this pattern:

1. **Type** (`lib/types.ts`): Add interface, add to `FinanceState`
2. **Constants** (`lib/constants.ts`): Add to `DEFAULT_STATE`, add nav item to `NAV_ITEMS`
3. **Supabase** (`lib/supabase.ts`): Add `toX`/`fromX` mappers (include `user_id` in `fromX`), CRUD functions (all accept `userId`), update `fetchAllData`
4. **Context** (`contexts/FinanceContext.tsx`): Add action types, reducer cases, `persistAction` cases
5. **Page** (`app/[section]/page.tsx`): Create page using `EditableTable` component
6. **Migration** (`migrations/`): Add numbered SQL file for the new table (include `user_id UUID REFERENCES finance_users(id)`)

### Dashboard Calculations

The dashboard (`app/page.tsx`) calculates:
- **Total Assets** = bank accounts + crypto
- **Total Debts** = credit card balances + personal debts (NOT family debts)
- **Net Debt** = max(0, debts - pending incoming)
- **Net Worth** = assets - net debt

Family debts, pet expenses, and family owed are intentionally excluded from all dashboard totals.

### Supabase Tables

| Table | Maps to | Notes |
|-------|---------|-------|
| `finance_users` | — | Auth table (Firebase UID linked) |
| `finance_accounts` | `Account` | Bank accounts + credit cards |
| `finance_debts` | `Debt` | Personal debts (with paid-off flag) |
| `finance_family_debts` | `FamilyDebt` | Excluded from dashboard totals |
| `finance_crypto_holdings` | `CryptoHolding` | ETH, USDC, GBPe |
| `finance_incomings` | `Incoming` | pending/received status |
| `finance_budget_line_items` | `BudgetLineItem` | Grouped by month, recurring support |
| `finance_annual_subscriptions` | `AnnualSubscription` | With renewal dates + account linking |
| `finance_pet_expenses` | `PetExpense` | Excluded from dashboard totals |
| `finance_family_owed` | `FamilyOwed` | Tracks amount/paid/remaining |
| `finance_wallet_addresses` | `WalletAddress` | For on-chain balance lookups |
| `finance_shared_categories` | — | Shared expense categories |
| `finance_shared_category_members` | — | Category membership |
| `finance_shared_items` | — | Shared items with receipt support |

DB uses snake_case, TypeScript uses camelCase. Mapping is handled by `toX`/`fromX` helpers in `supabase.ts`. All data tables have `user_id` FK for per-user isolation. RLS is enabled on all tables (migration 018).

### Styling Conventions

- Cards: `rounded-lg border border-zinc-200 bg-white p-5`
- Labels: `text-xs font-semibold text-zinc-700`
- Financial numbers: `font-mono` with `text-positive` / `text-negative`
- Sidebar: `bg-zinc-900` with active state `bg-zinc-800 text-white`
- Tables use the `.sheet` CSS class from `globals.css`
- UI primitives from shadcn/ui (`components/ui/`)
- Animated components from animate-ui (`components/animate-ui/`)

### Currencies

Supported: CAD, USD, GBP, EUR. Crypto: ETH, USDC, GBPe. Rates fetched on mount and cached in localStorage. Display currency toggleable by user.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gitpancake) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
