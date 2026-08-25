## myexpectpay

> This file provides guidance to Claude Code when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**MyExpertPay** is a full-stack payment management portal for Expertpay account holders. The service allows employers to pay employees via debit cards; this portal lets users view balances, manage bank accounts, manage child-support cases and recipients, and review payment history.

This is a greenfield rewrite of an existing React + Firebase app into a **Next.js + Firestore** monolith deployed on **Firebase App Hosting**. The goal is a production-quality application — prioritise correctness, security, and type safety.

## Tech Stack

### Framework
- **Next.js 14+** — App Router, TypeScript strict mode
- **React 18** — Server Components (default) + Client Components (`"use client"`) where interactivity is needed

### Frontend (within Next.js)
- **Routing:** Next.js App Router file-based routing
- **Server state:** TanStack Query (client components) + React Server Components (server reads)
- **UI state:** React Context or `useState` — no Zustand (App Router reduces global state needs)
- **Forms:** React Hook Form + **Zod** resolver (unified validation library)
- **UI components:** Shadcn/UI + custom CSS Modules
- **Styling:** Tailwind CSS + CSS Modules (`.module.css` per component)
- **Charts:** Recharts
- **i18n:** react-intl (EN, DE, ES)
- **Testing:** Vitest + React Testing Library

### Backend (Next.js Route Handlers + Server Actions)
- **API:** Next.js Route Handlers (`src/app/api/...`) — replaces Express
- **Database:** Firestore via **Firebase Admin SDK** (server-side only — no client SDK Firestore access)
- **Auth verification:** Firebase Admin SDK — verifies session cookie on every server request
- **Validation:** Zod (all Route Handler request bodies and params)
- **Testing:** Vitest + Firebase Emulator Suite (Firestore + Auth emulators)

### Infrastructure
- **Auth:** Firebase Authentication (Google OAuth + email/password)
- **Session management:** Firebase session cookies (server-side; `firebase-admin` creates/verifies via `auth().createSessionCookie`)
- **Database:** Firestore (Google-managed, encrypted at rest)
- **Hosting:** Firebase App Hosting (Next.js SSR/SSG/ISR via Cloud Run)
- **CI/CD:** GitHub Actions → Firebase App Hosting GitHub integration
- **Secrets:** Firebase App Hosting environment secrets (no `.env` in repo)

## Architecture

### Core User Flows

1. **Login** → Google OAuth or email/password → dashboard
2. **Dashboard** → account summary + payment chart + activity calendar + recent messages
3. **Bank Accounts** → list → add (routing + account number + type) → edit / verify / delete
4. **Cases** → list → add (case number + NCP + children[]) → edit / delete
5. **Recipients** → list → add (name + email + case) → edit / delete
6. **Payments** → list + filter (date range + status) → send money / request money
7. **Messages** → full inbox, read/unread state
8. **Profile** → edit display name, change password
9. **Settings** → language switcher, delete account

### Request Flow

```
[Browser]
  │
  ├─ Server Components (RSC)  →  Firebase Admin SDK  →  Firestore
  │
  └─ Client Components
       └─ fetch() / TanStack Query  →  Route Handlers (`/api/...`)
                                          └─ verify session cookie
                                          └─ Firebase Admin SDK  →  Firestore
```

### Auth Flow

```
[User] → [Firebase Auth client SDK (Google popup / email+pw)] → [ID Token]
       → [POST /api/auth/session]  →  Firebase Admin: verifyIdToken + createSessionCookie
       → [session cookie set on browser]
       → [middleware.ts verifies cookie on every protected route]
```

### Firestore Data Model

All user data lives in subcollections under `users/{uid}/` — this makes user scoping implicit and simple.

```
users/{uid}
  ├── bankAccounts/{bankId}
  │     fields: bankName, nickname, routingNumber (encrypted), accountNumber (encrypted),
  │             accountNumberLast4, accountType, verified, isPrimary,
  │             receivePayments, sendPayments, createdAt, updatedAt
  │
  ├── cases/{caseId}
  │     fields: caseNumber, ncpName, children (string[]), createdAt, updatedAt
  │
  ├── recipients/{recipientId}
  │     fields: firstName, lastName, email, caseId, createdAt, updatedAt
  │
  ├── payments/{paymentId}
  │     fields: amount, bankId, caseNumber, recipientId, recipientName,
  │             paymentDate, status, type, note, createdAt
  │
  └── messages/{messageId}
        fields: sender, subject, body, isRead, createdAt
```

### Project Structure

```
myexpectpay/
├── src/
│   ├── app/
│   │   ├── (auth)/                  ← unauthenticated layout group
│   │   │   ├── layout.tsx
│   │   │   ├── login/page.tsx
│   │   │   ├── register/page.tsx
│   │   │   └── forgot-password/page.tsx
│   │   ├── (dashboard)/             ← authenticated layout group
│   │   │   ├── layout.tsx           ← auth guard + shell nav
│   │   │   ├── page.tsx             ← dashboard
│   │   │   ├── bank-accounts/
│   │   │   ├── cases/
│   │   │   ├── recipients/
│   │   │   ├── payments/
│   │   │   ├── messages/
│   │   │   ├── profile/
│   │   │   └── settings/
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   │   ├── session/route.ts   ← POST: create session cookie
│   │   │   │   └── logout/route.ts    ← POST: clear session cookie
│   │   │   ├── banks/route.ts         ← GET list, POST create
│   │   │   ├── banks/[id]/route.ts    ← GET, PATCH, DELETE
│   │   │   ├── cases/route.ts
│   │   │   ├── cases/[id]/route.ts
│   │   │   ├── recipients/route.ts
│   │   │   ├── recipients/[id]/route.ts
│   │   │   ├── payments/route.ts
│   │   │   ├── payments/send/route.ts
│   │   │   ├── payments/request/route.ts
│   │   │   ├── messages/route.ts
│   │   │   ├── messages/[id]/read/route.ts
│   │   │   ├── users/me/route.ts      ← DELETE: delete account
│   │   │   └── dashboard/route.ts
│   │   └── layout.tsx               ← root layout (fonts, providers)
│   ├── components/
│   │   ├── layout/                  ← AppShell, Header, Nav, Footer
│   │   └── ui/                      ← Modal, Spinner, Pagination, Toast, EmptyState, Icon
│   ├── lib/
│   │   ├── firebase/
│   │   │   ├── admin.ts             ← Firebase Admin SDK singleton (server-only)
│   │   │   └── client.ts            ← Firebase client SDK (auth only, "use client")
│   │   ├── firestore/               ← data access layer (one file per collection)
│   │   │   ├── banks.ts
│   │   │   ├── cases.ts
│   │   │   ├── recipients.ts
│   │   │   ├── payments.ts
│   │   │   └── messages.ts
│   │   └── session.ts               ← getSession() helper (reads cookie, returns uid)
│   ├── middleware.ts                 ← route protection (verify session cookie)
│   ├── hooks/                       ← client-side hooks only
│   ├── translations/                ← en.json, de.json, es.json
│   ├── types/                       ← shared TypeScript interfaces
│   ├── constants.ts                 ← PAGE_SIZE, ABA regex, limits
│   └── utils/                       ← formatMoney, validateRouting, formatDate
├── firebase.json
├── apphosting.yaml                  ← Firebase App Hosting build/run config
├── firestore.rules                  ← deny all (Admin SDK bypasses rules)
├── firestore.indexes.json
└── REQUIREMENTS.md
```

## Key Requirements to Keep in Mind

- **User scoping:** every Firestore read/write must use `users/{uid}/...` — never query across users without admin context
- **Input validation on both sides:** Zod on the form (via React Hook Form resolver) AND Zod in the Route Handler
- **All destructive actions need a confirmation dialog** (delete bank, case, recipient, account)
- **Routing number validation:** ABA checksum algorithm — validate in `src/utils/validateRouting.ts`, called from both form schema and API route schema
- **Account number:** display last 4 digits only; store full number encrypted at the application layer before writing to Firestore (Google encrypts Firestore at rest but not at the field level)
- **Payment status** is an enum (10 values); define it once in `src/types/` and import everywhere
- **Pagination:** all Route Handlers default to 20 items per page; all list UIs use the shared `<Pagination>` component; cursor-based pagination preferred for Firestore (use `startAfter`)
- **i18n:** all user-facing strings go through `react-intl` — no hardcoded English strings in components
- **Server vs Client Components:** default to Server Components; only add `"use client"` when you need `useState`, `useEffect`, event handlers, or browser APIs. Forms need `"use client"` for React Hook Form.
- **Session cookie lifetime:** 5 days; refresh on activity; revoke on logout via `auth().revokeRefreshTokens(uid)`

## Development Commands

```bash
# Install
npm install

# Dev server (Next.js + Firebase Emulators)
npm run dev              # Next.js dev server (http://localhost:3000)
npm run emulators        # Firebase Emulators (Firestore + Auth)
npm run dev:full         # Both concurrently

# Build & lint
npm run build
npm run lint
npm run type-check

# Test
npm run test             # Vitest unit tests
npm run test:e2e         # Playwright E2E

# Firebase
firebase deploy          # deploy to App Hosting
firebase emulators:start # Firestore + Auth emulators locally
```

## Environment Variables

Set via Firebase App Hosting secrets (never in `.env` files committed to the repo).

```
# Server-side (Firebase Admin)
FIREBASE_PROJECT_ID
FIREBASE_CLIENT_EMAIL
FIREBASE_PRIVATE_KEY

# Client-side (public — safe to expose)
NEXT_PUBLIC_FIREBASE_API_KEY
NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN
NEXT_PUBLIC_FIREBASE_PROJECT_ID
NEXT_PUBLIC_FIREBASE_APP_ID

# App
ACCOUNT_ENCRYPTION_KEY    ← AES-256 key for encrypting account numbers
SESSION_COOKIE_SECRET     ← used to sign/verify session cookies
```

## Working the design-gap backlog (issues #41–#59)

These 19 issues track UI gaps found by comparing the build against the `frontend/` reference
(see `doc/CODE_REVIEW.md` and issue #38). They are grouped under the GitHub milestone
**"Design-gap backlog (#41–#59)"**, which is the work queue.

**Flow: one issue → one branch → one PR that closes it → stop for review.** Never merge; never
work more than one issue per run. Each new issue branches off the latest `main` (assumes prior
gap PRs are merged during the review pause); if the target shares files with a still-open gap
PR, flag it before proceeding.

Per-issue recipe: read the `frontend/` reference + our implementation → branch → implement
(reuse components/tokens/i18n; new APIs follow the `getSession` + Zod + `firestore/*` pattern)
→ `npm run type-check` + `npm run test` (must pass) → before/after Playwright screenshots
(test user `bb@gmail.com`, `POST /api/seed` for data) → log note in P04-012 → PR with
`Closes #<n>`.

The step-by-step is encoded in the local slash command `/gap-issue` (`.claude/commands/gap-issue.md`,
git-ignored / local-only). Run `/gap-issue [number]` to work the next (or a specific) issue.

---
> Source: [sheetstone/myexpectpay](https://github.com/sheetstone/myexpectpay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
