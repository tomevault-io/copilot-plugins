## money-flow

> > **For Gemini CLI**: This file provides essential context for the Money Flow 3 project when using Gemini CLI.

# Money Flow 3 - Gemini CLI Context

> **For Gemini CLI**: This file provides essential context for the Money Flow 3 project when using Gemini CLI.

---

## 🎯 Project Overview

**Name**: Money Flow 3  
**Type**: Personal Finance Management Application  
**Tech Stack**: Next.js 15 (App Router), TypeScript, Tailwind CSS, Shadcn UI, Supabase (PostgreSQL)

### Core Features
- **Transactions**: Income, Expense, Transfer, Debt, Repayment tracking
- **Accounts**: Bank accounts, credit cards, wallets management
- **People**: Debt tracking with individuals
- **Services**: Recurring subscriptions (Netflix, Spotify, etc.)
- **Cashback**: Credit card rewards tracking
- **Installments**: Payment plans
- **Batch Processing**: Import from Excel/CSV

---

## 📚 Essential Documentation

### Must Read First
1. **`.agent/knowledge/root-docs/ONBOARDING.md`** - Master onboarding guide
2. **`.agent/rules/rules.md`** - Core coding standards
3. **`.agent/rules/ui_rules.md`** - UI/UX standards (STRICT)
4. **`.agent/CASHBACK_WORKFLOW.md`** - Advanced cashback rules & logic (CRITICAL for credit cards)

### Current Phase
- **Phase 15**: COMPLETED - Categories UI Optimization & Cashback Logic Refinement
- **Phase 16**: IN PROGRESS - Credit Card Advance & Profit Tracking
- See `.agent/HANDOVER_PHASE_16_CREDIT_PROFIT.md` for latest changes.

---

## 🔑 Key Concepts

### 1. Single Source of Truth
- All financial data lives in the `transactions` table
- `final_price` is auto-calculated via database trigger
- `transaction_history` stores edit snapshots

### 2. Transaction Types
- **Income**: Money received
- **Expense**: Money spent
- **Transfer**: Move between accounts
- **Debt**: Money lent to others
- **Repayment**: Money received back from debts

### 3. Unified Transaction Table
- **Component**: `src/components/moneyflow/unified-transaction-table.tsx`
- **Single source of truth** for transaction display
- **Single Flow Mode**: Simple Income/Expense shows Type Badge + Category/Shop (no FROM/TO badges)
- **Multi-Flow Mode**: Transfer/Debt shows detailed flow with FROM/TO badges

---

## 🏗️ Project Structure

```
money-flow-3/
├── src/
│   ├── app/                    # Next.js App Router pages
│   │   ├── accounts/           # Accounts management
│   │   ├── people/             # People/Debt management
│   │   ├── transactions/       # Transaction views
│   │   ├── services/           # Recurring services
│   │   └── batch/              # Batch import
│   ├── components/             # React components
│   │   ├── ui/                 # Shadcn UI components
│   │   ├── accounts/           # Account components
│   │   ├── people/             # People components
│   │   └── moneyflow/          # Transaction components
│   ├── actions/                # Server Actions
│   ├── services/               # Business logic layer
│   ├── lib/                    # Utilities, Supabase clients
│   └── types/                  # TypeScript types
├── .agent/                     # Documentation & guides
│   ├── prompts/                # Agent prompts
│   ├── rules/                  # Coding standards
│   └── archive/                # Old documentation
└── supabase/migrations/        # Database migrations
```

---

## 📋 Coding Standards (CRITICAL)

### Type Safety
- **NO `any` types**. Use defined types from `src/types/` or `database.types.ts`
- Always import types explicitly

### Server Actions
- Use for mutations (POST/PUT/DELETE)
- Always call `revalidatePath` after updates
- Wrap in try-catch blocks
- Return `{ success: boolean, error?: string, data?: T }`

### React Server Components
- Default to RSC
- Use `'use client'` only when needed (hooks, event listeners)

### Supabase
- Use `createClient` from `src/lib/supabase/server.ts` for Server Components/Actions
- Use `createClient` from `src/lib/supabase/client.ts` for Client Components
- Always assume RLS is enabled

---

## 🎨 UI Standards (STRICT)

### Avatars & Images
- **Square Avatars**: Images in dropdowns/lists/sidebar MUST use `rounded-none`.
- **No Cropped Images**: Documentation images must be full, uncropped.
- **Logo Fallback**: Income transactions use Bank/Account logo as fallback if shop image is missing.

### Typography
- **NO Monospace Fonts**: Never use `font-mono` for UI text (only code blocks)

### Flow Column
- **Single source of truth**: Always use `UnifiedTransactionTable`
- **No custom Flow UI** in detail pages
- **Single Flow Mode**: Type Badge + Category/Shop only
- **Multi-Flow Mode**: FROM/TO badges for transfers/debts

---

## 🚀 Development Workflow

### Setup
```bash
npm install
npm run dev
```

### Before Committing
- Ensure dev server runs without errors
- No TypeScript errors in IDE
- Follow UI rules (square avatars, no monospace fonts)
- Server Actions have error handling

### Commit Format
```
<type>: <description>

<detailed description>
```

**Types**: `fix`, `feat`, `chore`, `docs`, `refactor`

---

## 🐛 Known Issues

### iCloud Drive EPERM Error
- **Issue**: `npm run build` fails with EPERM error on `node_modules`
- **Cause**: Project is in iCloud Drive folder
- **Workaround**: Use dev server and TypeScript IDE checks
- **Status**: Known limitation, does not affect development

---

## 📖 Common Tasks

### Modifying Transaction Table
1. **ALWAYS** edit `src/components/moneyflow/unified-transaction-table.tsx`
2. **NEVER** create custom Flow UI in detail pages
3. Follow "Single Flow" logic for simple transactions
4. Test on both main `/transactions` page and detail pages

### Adding a New Field to Transactions
1. Update schema in `src/types/moneyflow.types.ts`
2. Modify Transaction Slide V2 (`src/components/transaction/slide-v2/`)
3. Update `src/services/transaction.service.ts`
4. Test thoroughly

### Adding a New Page
1. Create in `src/app/[page-name]/page.tsx`
2. Use React Server Components by default
3. Fetch data using Server Actions or direct Supabase calls
4. Follow existing page patterns

---

## 🆘 Getting Help

### When Stuck
1. Check `.agent/` documentation
2. Review code comments
3. Check git history: `git log`
4. Read handover docs: `.agent/HANDOVER_PHASE_*.md`

### Common Questions
- **"Where is transaction display logic?"** → `src/components/moneyflow/unified-transaction-table.tsx`
- **"How to add a new transaction field?"** → Update types, Slide V2, service layer
- **"Why is build failing?"** → iCloud Drive EPERM (known limitation)
- **"What are the UI rules?"** → `.agent/rules/ui_rules.md`

---

## ✅ Quick Checklist

Before starting work:
- [ ] Read `.agent/knowledge/root-docs/ONBOARDING.md`
- [ ] Read `.agent/rules/rules.md`
- [ ] Read `.agent/rules/ui_rules.md`
- [ ] Read current phase handover (`.agent/HANDOVER_PHASE_*.md`)
- [ ] Understand "Single Source of Truth" concept
- [ ] Know where transaction display logic lives
- [ ] Dev environment set up (`npm run dev` works)

---

**Last Updated**: 2026-02-12  
**Version**: 1.1  
**Maintained By**: Money Flow 3 Team

---

**For more details**: See `.agent/knowledge/root-docs/ONBOARDING.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rei6688)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rei6688)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
