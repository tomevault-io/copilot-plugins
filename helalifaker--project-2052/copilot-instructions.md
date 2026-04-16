## project-2052

> - Financial planning tool for 30-year projections (2023-2053)

# Project Zeta / Project_2052 - Cursor Rules
## Financial Planning Application

**Project Context:**
- Financial planning tool for 30-year projections (2023-2053)
- Millions of SAR in calculations
- Board-level decision support
- Three distinct calculation periods (Historical, Transition, Dynamic)
- Circular financial dependencies

**Tech Stack:**
- Next.js 16 (App Router)
- React 19
- TypeScript 5.7 (strict mode required)
- Prisma (PostgreSQL)
- Decimal.js (for all financial calculations)
- Zod (runtime validation)
- Vitest (testing)

---

## CRITICAL RULES - NEVER VIOLATE

### 1. Financial Calculations - ZERO TOLERANCE

**ALWAYS use Decimal.js for money. NEVER use JavaScript numbers.**

```typescript
// ✅ CORRECT
import Decimal from 'decimal.js';
const revenue = new Decimal('125300000');
const rent = revenue.times(0.08);
const netIncome = revenue.minus(rent);

// ❌ WRONG - Will cause precision errors
const revenue = 125300000;
const rent = revenue * 0.08; // Precision loss!
```

**Pre-create Decimal constants (performance critical):**
```typescript
// ✅ CORRECT - Pre-create constants
export const DECIMAL_ZERO = new Decimal(0);
export const DECIMAL_ONE = new Decimal(1);
export const ZAKAT_RATE = new Decimal(0.025);

// ❌ WRONG - Don't recreate in loops
function calculateZakat(ebt: Decimal): Decimal {
  const zakatRate = new Decimal(0.025); // Created every call - wasteful!
  return ebt.times(zakatRate);
}
```

**Compare Decimals using methods, not operators:**
```typescript
// ✅ CORRECT
if (cash.greaterThanOrEqualTo(minCash)) { ... }
if (ebt.lessThanOrEqualTo(DECIMAL_ZERO)) { ... }

// ❌ WRONG
if (cash >= minCash) { ... } // Type error + precision loss
if (ebt.toNumber() <= 0) { ... } // Loses precision
```

### 2. Type Safety - MANDATORY

**Never use `any` type. Always type explicitly.**

```typescript
// ✅ CORRECT
function calculateRent(
  baseRent: Decimal,
  growthRate: number,
  years: number
): Decimal {
  return baseRent.times(Math.pow(1 + growthRate, years));
}

// ❌ WRONG
function calculateRent(baseRent: any, growthRate: any): any {
  return baseRent * (1 + growthRate);
}
```

**Use Zod for runtime validation:**
```typescript
// ✅ CORRECT
import { z } from 'zod';
const ProposalSchema = z.object({
  name: z.string().min(1),
  baseRent2028: z.number().positive(),
  rentModel: z.enum(['FIXED', 'REVSHARE', 'PARTNER'])
});

// ❌ WRONG
const proposal = await request.json(); // No validation!
```

### 3. API Routes - Security & Validation Required

**Every API route MUST:**
1. Authenticate and authorize (RBAC)
2. Validate input with Zod
3. Handle errors properly
4. Return appropriate status codes

```typescript
// ✅ CORRECT
export async function POST(request: NextRequest) {
  try {
    // 1. Authenticate & authorize
    const user = await requireAuth(request, [Role.ADMIN, Role.PLANNER]);
    
    // 2. Parse & validate
    const body = await request.json();
    const validated = CreateProposalSchema.parse(body);
    
    // 3. Business logic
    const proposal = await db.leaseProposal.create({
      data: { ...validated, createdBy: user.id }
    });
    
    return NextResponse.json(proposal, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: 'Validation failed', details: error.errors },
        { status: 400 }
      );
    }
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}

// ❌ WRONG - No auth, no validation, no error handling
export async function POST(request: NextRequest) {
  const body = await request.json();
  const proposal = await db.leaseProposal.create({ data: body });
  return NextResponse.json(proposal);
}
```

### 4. React & Next.js - Performance & Best Practices

**Server Components by default. 'use client' only when needed.**

```typescript
// ✅ CORRECT - Server Component (default)
// app/proposals/[id]/page.tsx
export default async function ProposalPage({ params }: { params: { id: string } }) {
  const proposal = await db.leaseProposal.findUnique({
    where: { id: params.id }
  });
  return <ProposalDetail proposal={proposal} />;
}

// ✅ CORRECT - Client Component (only when needed)
// components/ProposalDetail.tsx
'use client';
export function ProposalDetail({ proposal }: { proposal: Proposal }) {
  const [activeTab, setActiveTab] = useState('overview');
  return <Tabs value={activeTab} onValueChange={setActiveTab}>...</Tabs>;
}
```

**Memoize expensive computations:**
```typescript
// ✅ CORRECT
const FinancialTable = memo(({ data }: { data: FinancialPeriod[] }) => {
  const formattedData = useMemo(() => {
    return data.map(period => ({
      year: period.year,
      revenue: formatMillions(period.revenue),
      rent: formatMillions(period.rent)
    }));
  }, [data]);
  return <Table data={formattedData} />;
});

// ❌ WRONG - Recalculates every render
function FinancialTable({ data }: { data: FinancialPeriod[] }) {
  const formattedData = data.map(period => formatMillions(period.revenue));
  return <Table data={formattedData} />;
}
```

**Debounce interactive inputs (300ms):**
```typescript
// ✅ CORRECT
import { useDebouncedCallback } from 'use-debounce';
const debouncedRecalculate = useDebouncedCallback(
  (value: number) => recalculate({ enrollmentPercent: value }),
  300
);

// ❌ WRONG - Fires on every pixel movement
const handleSliderChange = (value: number) => {
  recalculate({ enrollmentPercent: value }); // Too many calls!
};
```

**Use Web Workers for heavy calculations:**
```typescript
// ✅ CORRECT - 30-year calculations in worker
// src/workers/calculation.worker.ts
self.onmessage = (e: MessageEvent<ProposalInputs>) => {
  const result = calculateFinancials(e.data);
  self.postMessage(serializeFinancials(result));
};

// ❌ WRONG - Blocks UI thread
function MyComponent() {
  const handleCalculate = (inputs: ProposalInputs) => {
    const financials = calculateFinancials(inputs); // Blocks UI!
    setResult(financials);
  };
}
```

### 5. Database & Prisma

**Select only needed fields:**
```typescript
// ✅ CORRECT
const proposals = await db.leaseProposal.findMany({
  select: { id: true, name: true, rentModel: true },
  where: { createdBy: userId }
});

// ❌ WRONG - Fetches all fields
const proposals = await db.leaseProposal.findMany();
```

**Use transactions for multi-table operations:**
```typescript
// ✅ CORRECT
await db.$transaction(async (tx) => {
  const proposal = await tx.leaseProposal.create({ data: proposalData });
  await tx.auditLog.create({ data: logData });
});

// ❌ WRONG - Not atomic
const proposal = await db.leaseProposal.create({ data: proposalData });
await db.auditLog.create({ data: logData }); // Not atomic!
```

**Convert Prisma.Decimal ↔ Decimal.js properly:**
```typescript
// ✅ CORRECT
import { Prisma } from '@prisma/client';
import Decimal from 'decimal.js';

// Save
await db.systemConfig.update({
  data: { zakatRate: new Prisma.Decimal(zakatRate.toString()) }
});

// Read
const config = await db.systemConfig.findUnique({ where: { id } });
const zakatRate = new Decimal(config.zakatRate.toString());

// ❌ WRONG - Precision loss
await db.systemConfig.update({
  data: { zakatRate: zakatRate.toNumber() } // Loses precision!
});
```

**Schema design:**
- Use UUID for IDs (never Int)
- Use enums (never String for fixed values)
- Add indexes on foreign keys and frequently queried fields
- Include createdAt/updatedAt timestamps

### 6. Testing Requirements

**Minimum coverage:**
- Calculation engine: 100% (financial accuracy critical)
- API routes: 90%
- React components: 80%
- Utilities: 90%
- Overall: >85%

**Test financial calculations thoroughly:**
```typescript
// ✅ CORRECT
describe('Fixed Rent Model', () => {
  it('calculates base year rent correctly', () => {
    const result = calculateFixedRent({
      baseRent2028: new Decimal(10_000_000),
      growthRate: 0.05,
      frequency: 1
    });
    expect(result[0]).toEqual(new Decimal(10_000_000));
  });
});
```

### 7. Error Handling

**Use custom error classes:**
```typescript
// ✅ CORRECT
export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(`${resource} not found`, 404, 'NOT_FOUND');
  }
}

if (!proposal) {
  throw new NotFoundError('Proposal');
}
```

**Consistent error response format:**
```typescript
interface ErrorResponse {
  error: string;
  code?: string;
  details?: unknown;
  timestamp: string;
}
```

### 8. Code Organization

**Directory structure:**
```
src/
├── app/                    # Next.js App Router
│   ├── api/               # API routes
│   └── [pages]/           # Page routes
├── components/            # React components
│   ├── ui/               # shadcn/ui base components
│   ├── financial/        # Financial-specific components
│   └── forms/            # Form components
├── lib/                   # Core libraries
│   ├── engine/           # Calculation engine
│   ├── validation/       # Zod schemas
│   ├── formatting/       # Display formatting
│   └── auth/             # Authentication
├── hooks/                 # Custom React hooks
├── types/                 # TypeScript types
└── workers/               # Web Workers
```

**Import order:**
1. External dependencies
2. Internal absolute imports (grouped by category)
3. Relative imports

### 9. Documentation

**Document complex logic with JSDoc:**
```typescript
/**
 * Calculates fixed escalation rent for 26 years (2028-2053).
 *
 * @param params - Fixed rent calculation parameters
 * @param params.baseRent2028 - Base rent in SAR for year 2028
 * @param params.growthRate - Annual growth rate as decimal (e.g., 0.05 = 5%)
 * @returns Array of 26 Decimal values representing rent for each year
 */
export function calculateFixedRent(params: FixedRentParams): Decimal[] {
  // Implementation
}
```

---

## TOP 10 RULES (Memorize)

1. ✅ **Decimal.js for ALL money calculations** - Never use JavaScript numbers
2. ✅ **Pre-create Decimal constants** - Don't recreate in loops
3. ✅ **Explicit TypeScript types** - Never use `any` type
4. ✅ **Validate all API inputs** - Use Zod on every API route
5. ✅ **Memoize React computations** - Use memo, useMemo, useCallback
6. ✅ **Debounce interactive inputs** - 300ms for sliders and search
7. ✅ **Server Components by default** - 'use client' only when needed
8. ✅ **RBAC on all protected routes** - requireAuth([Role.ADMIN, Role.PLANNER])
9. ✅ **>80% test coverage minimum** - Financial code needs 100%
10. ✅ **Document complex logic** - JSDoc + inline comments

---

## CRITICAL "NEVER DO" LIST

❌ **NEVER use JavaScript numbers for financial calculations**
❌ **NEVER skip input validation on API routes**
❌ **NEVER skip authentication/authorization**
❌ **NEVER use `any` type in TypeScript**
❌ **NEVER skip memoization in React**
❌ **NEVER use 'use client' unnecessarily**
❌ **NEVER skip transactions for multi-table operations**
❌ **NEVER use Int for database IDs (use UUID)**
❌ **NEVER skip error handling**
❌ **NEVER ship code with <80% test coverage**

---

## AGENT ROLES & RESPONSIBILITIES

**Financial Architect (fa-001):**
- Critical: Section 3 (Financial Calculations) - 100% compliance
- Zero tolerance: JavaScript numbers, no Decimal constants
- Test coverage: 100% for financial code

**Backend Engineer (be-001):**
- Critical: API validation, Security (RBAC), Error handling
- Zero tolerance: No validation, no RBAC, raw SQL
- Test coverage: 90% for API routes

**Frontend Engineer (fe-001):**
- Critical: Performance (memoization), React/Next.js patterns
- Zero tolerance: 'use client' everywhere, no memoization
- Test coverage: 80% for components

**Database Architect (da-001):**
- Critical: Schema design, Query optimization
- Zero tolerance: Int IDs, no indexes, fetch all fields
- Review: Every schema change

**QA Engineer (qa-001):**
- Critical: Testing standards, All sections for validation
- Zero tolerance: Approving code <80% coverage
- Review: Every PR + all phase gates

**Project Manager (pm-001):**
- Critical: ALL sections (enforces compliance)
- Zero tolerance: Approving violations
- Review: Every PR, every handoff, every gate

---

## ENFORCEMENT

**6 Layers of Enforcement:**
1. Documentation (CODING_STANDARDS.md)
2. Pre-Commit Hooks (lint, type-check, test)
3. Code Review (every PR reviewed)
4. CI/CD Pipeline (automated gates)
5. Weekly Audits (PM runs compliance audit)
6. Phase Gate Certification (QA certifies compliance)

**Violation Consequences:**
- First: PR rejected, fix required
- Second: Pair programming session
- Third: Escalation to CAO, potential reassignment

**Critical violations escalate immediately:**
- Using JavaScript numbers for money
- Skipping authentication/authorization
- No input validation on API endpoints
- Shipping code with <50% test coverage
- Using `any` type in financial calculations

---

## PERFORMANCE TARGETS

- **30-year calculation:** < 1 second
- **API response time:** < 200ms average
- **Test coverage:** >80% overall, 100% for financial code
- **Calculation accuracy:** 100% (zero tolerance for errors)

---

## KEY PROJECT DOCUMENTS

**Must Read:**
- `CODING_STANDARDS.md` - Complete technical reference (1686 lines)
- `CODING_STANDARDS_ENFORCEMENT.md` - Enforcement policy
- `README_CODING_STANDARDS.md` - Quick start guide
- `AGENT_IMPLEMENTATION_GUIDE.md` - Agent orchestration system
- `STACK_DOCUMENTATION.md` - **Library documentation and usage patterns** (consult when implementing features with Next.js, Prisma, Supabase, React, Zod, Zustand, TanStack Table, or Recharts)

**For Financial Work:**
- `04_FINANCIAL_RULES.md` - Every calculation rule, formulas, business logic

**For Technical Implementation:**
- `03_TSD_COMPREHENSIVE.md` - Technical architecture, stack decisions

**For Design Work:**
- `06_UI_UX_SPECIFICATION.md` - Design philosophy, color palette, UX patterns

**For Requirements:**
- `01_BCD.md` - Business context
- `02_PRD.md` - Complete product requirements, user stories

---

## WHEN IN DOUBT

1. **Refer to CODING_STANDARDS.md** - Your complete reference
2. **Consult STACK_DOCUMENTATION.md** - When implementing features with:
   - Next.js (App Router, Server Components, API routes, data fetching)
   - Prisma (queries, migrations, schema design)
   - Supabase (client setup, authentication, realtime)
   - React (hooks, Server Components, state management)
   - Zod (schema validation, parsing)
   - Zustand (store creation, persistence)
   - TanStack Table (table setup, sorting, filtering, pagination)
   - Recharts (chart components, responsive containers)
3. **Ask PM** - Don't guess, don't skip, don't compromise
4. **Review examples** - 100+ code examples in CODING_STANDARDS.md and STACK_DOCUMENTATION.md
5. **Check agent-specific requirements** - See agent roles above

---

**Remember: This is a financial planning tool for millions of SAR in decisions. Financial accuracy, security, and performance are non-negotiable. Follow the standards. Build something excellent.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/helalifaker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
