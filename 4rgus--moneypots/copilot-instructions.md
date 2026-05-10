## moneypots

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Start Vite dev server at http://localhost:5173
npm run build     # Production build → ./dist/
npm run preview   # Preview production build locally
npx tsc --noEmit  # Type-check without emitting files
```

No test runner, linter, or formatter is configured.

## Environment Setup

Copy `.env.example` to `.env.local` and fill in the Firebase config values:

```
VITE_FIREBASE_API_KEY
VITE_FIREBASE_AUTH_DOMAIN
VITE_FIREBASE_PROJECT_ID
VITE_FIREBASE_STORAGE_BUCKET
VITE_FIREBASE_MESSAGING_SENDER_ID
VITE_FIREBASE_APP_ID
```

All env vars are prefixed with `VITE_` and accessed via `import.meta.env.*`. Types are declared in [src/vite-env.d.ts](src/vite-env.d.ts).

## Architecture

**Money Pots** is an envelope-budgeting SPA. Users allocate real bank account balances into virtual saving pots. Authentication is Google OAuth only. All data is stored per-user in Firestore with real-time subscriptions.

The project is written in **TypeScript** (migrated from JavaScript). All source files use `.ts`/`.tsx` extensions. Shared domain types live in [src/types.ts](src/types.ts).

### Core mental model

Account balances are **reference values** — they are never mutated by pot top-ups. Adding money to a pot creates a `transaction` record. The committed amount per account is always derived at render time from transactions, not stored. This prevents double-counting when pots are reset and re-topped.

### Data flow

```
Firebase onSnapshot subscriptions → Dashboard.tsx state → tab components → Firestore writes
```

`Dashboard.tsx` sets up three live subscriptions (pots, accounts, transactions) and passes data + write handlers down as props. There is no global state library — Firestore is the source of truth.

`Dashboard.tsx` also computes `effectiveAccounts` (accounts with balance reduced by committed pot amounts) and splits pots into active vs completed for the respective tabs.

### Key library files

| File | Purpose |
|------|---------|
| [src/types.ts](src/types.ts) | Shared TypeScript interfaces: `Pot`, `Account`, `EffectiveAccount`, `Transaction`, `TransactionSource`, `Alloc`, `AccountHistoryEntry`, `AccountHistoryType` |
| [src/lib/firebase.ts](src/lib/firebase.ts) | Firebase app init, Google auth provider |
| [src/lib/db.ts](src/lib/db.ts) | All Firestore CRUD + `onSnapshot` subscriptions |
| [src/lib/allocation.ts](src/lib/allocation.ts) | Smart allocation algorithm, committed calculation, account contributions |
| [src/hooks/useAuth.ts](src/hooks/useAuth.ts) | `user`, `login()`, `logout()` from Firebase auth |

### Allocation algorithm (`src/lib/allocation.ts`)

**`smartAllocate(accounts, amount)`** distributes a requested amount across accounts greedily by highest available balance first.

- **Salary accounts:** full balance is available
- **Savings accounts:** available = balance − minBalance
- Returns `{ allocs, shortfall }` — shortfall > 0 if accounts can't cover the amount

**`computeCommitted(pots, transactions)`** walks transactions newest-first and returns `{ [accountId]: number }` — the total amount committed to active pots per account. Stops at each pot's current `saved` value to avoid double-counting from reset+retopup cycles.

**`getAccountContributions(accountId, pots, transactions)`** is like `computeCommitted` but scoped to one account. Returns `{ contributions: { [potId]: number }, order: string[] }` — used by the deficit resolution flow.

**`getPotContributions(pot, transactions)`** is the inverse — scoped to one pot, returns `{ [accountId]: number }` showing how much each account contributed to that pot's current `saved` amount. Used by the fulfill-goal flow.

**`fmt(n)`** formats numbers as Indian Rupee strings (₹, `en-IN` locale).

### `effectiveAccounts`

Computed in `Dashboard.tsx` via `useMemo`:

```ts
accounts.map(acc => ({
  ...acc,
  rawBalance: acc.balance,          // original user-entered value
  balance: Math.max(0, acc.balance - committed[acc.id]),  // available for allocation
}))
```

`effectiveAccounts` is passed to `PotsTab` and `AccountsTab`. Raw `accounts` are passed to `OverviewTab` and `HistoryTab`.

### Tabs

| Tab | Component | Description |
|-----|-----------|-------------|
| Overview | `OverviewTab` | Spending summary, goals breakdown, accounts breakdown |
| Pots | `PotsTab` | Active pots (saved < target). Completed pots show a 3-second celebration then move to Completed tab |
| Completed | `CompletedTab` | Pots where saved ≥ target. Allows **Mark as Spent** (deducts from source accounts + deletes pot), reset (saved → 0), or delete |
| Accounts | `AccountsTab` | Account cards with committed/free amounts. Deficit resolution on balance edit. Expandable per-account balance history panel |
| History | `HistoryTab` | Transaction log |

### Pot deadlines

Pots have an optional `deadline` (ISO date string). When set:

- **Fixed daily/monthly rate** = `ceil(target / totalDays)` where `totalDays` = createdAt → deadline duration. This is fixed — it does not change as the user adds money.
- **On-track status** = `saved >= (elapsed / totalDays) * target`
- **Catch-up amount** = `ceil(expectedSaved - saved)` when behind
- Displayed in PotsTab as a chip with a rate toggle (per month ↔ per day)
- Quick-fill buttons in TopUpModal: Monthly target, Daily target, Catch up

### Deficit resolution

When a user edits an account balance to less than the committed amount, `AccountsTab` detects the deficit and shows `DeficitModal` with three options:

- **A — Warning only:** save new balance, show deficit badge on account card, no pot changes
- **B — Proportional:** reduce each affected pot's `saved` proportionally based on its share
- **C — Newest first:** absorb deficit from most recently funded pot first

### Account balance history

Every balance change is logged to `users/{uid}/accounts/{accId}/history/` with a reason. This gives an audit trail without changing the core model (balance is still stored as an absolute value on the account document).

`AccountModal` shows a reason dropdown when editing (not when adding):
- **Salary / Deposit** → `credit`
- **Expense / Withdrawal** → `debit`
- **Sync with bank** → `correction`
- **Goal fulfilled** → `goal_fulfilled` (written automatically by the fulfill flow, not selectable by user)

`AccountsTab` has a "History" toggle per account card that opens an inline panel with the 20 most recent entries (live `onSnapshot` subscription, scoped per account).

### Fulfill goal flow

When a completed pot is marked as spent ("Mark as Spent" in `CompletedTab`):
1. `getPotContributions` determines which accounts funded the pot and by how much
2. `FulfillModal` shows the breakdown for confirmation
3. On confirm: `updateAccountWithHistory` reduces each source account's balance and writes a `goal_fulfilled` history entry; `deletePot` removes the pot
4. Net effect: `effectiveBalance` stays the same (committed removed, raw balance reduced by equal amount) but now correctly reflects reality

### Firestore schema

All collections live under `users/{userId}/`:

- **pots:** `{ name, target, saved, icon (emoji), color (hex), deadline (ISO string | null), createdAt }`
- **accounts:** `{ name, bank, type ('salary'|'savings'), balance, minBalance, createdAt }`
- **accounts/{accId}/history:** `{ type ('credit'|'debit'|'correction'|'goal_fulfilled'), delta (signed), newBalance, note (optional), createdAt }`
- **transactions:** `{ potId, potName, amount, sources: [{accountId, accountName, deduct}], createdAt }`

Security rules in `firestore.rules` ensure each user can only access their own data. The `history` subcollection needs an explicit rule added if rules are path-specific (not wildcard).

### Styling

Pure CSS with custom properties (no framework). Design tokens are in [src/index.css](src/index.css):

- `--clay` / `--clay-dark` / `--clay-light` / `--clay-pale` — primary warm brown palette
- `--sand` — body background
- `--ink` / `--ink-muted` / `--ink-faint` — text hierarchy
- `--border` / `--border-med` — border colours

Fonts: **DM Sans** (body) and **Fraunces** serif (headings/amounts).

### Deployment

`vercel.json` rewrites all routes to `/index.html` for client-side routing. After deploying to Vercel, add the Vercel domain to Firebase's authorized domains list.

---
> Source: [4RGUS/MoneyPots](https://github.com/4RGUS/MoneyPots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
