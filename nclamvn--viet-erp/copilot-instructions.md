## viet-erp

> > **This file configures Claude Code behavior for the VietERP MRP project.**

# CLAUDE.md - VietERP MRP AI Kernel Configuration

> **This file configures Claude Code behavior for the VietERP MRP project.**
> **Version:** 1.0.0 | **Last Updated:** 01/01/2026

---

## 🎯 PROJECT IDENTITY

```yaml
project: VietERP MRP (Material Requirements Planning)
type: Full-stack Manufacturing Intelligence System
language: TypeScript
framework: Next.js 15 (App Router)
```

## 🏗️ TECH STACK

| Layer | Technology |
|-------|------------|
| Frontend | React 19, TypeScript 5.x, Tailwind CSS, Shadcn/UI, Framer Motion |
| Backend | Next.js API Routes, Prisma ORM, PostgreSQL/SQLite |
| Auth | NextAuth.js 5 |
| AI | Google Gemini, OpenAI (fallback), Vercel AI SDK |
| Cache | Redis (in-memory fallback) |
| Queue | BullMQ pattern (in-memory) |

## 📁 PROJECT STRUCTURE

```
src/
├── app/                    # Next.js App Router pages
│   ├── (auth)/            # Auth routes (login, register)
│   ├── (dashboard)/       # Protected dashboard routes
│   └── api/               # API routes
├── components/            
│   ├── ui/                # Shadcn/UI components
│   ├── forms/             # Form components
│   └── [feature]/         # Feature-specific components
├── lib/                   
│   ├── ai/                # AI integration (Gemini, OpenAI)
│   ├── cache/             # Caching utilities
│   ├── security/          # Rate limiting, auth
│   └── utils/             # Shared utilities
├── hooks/                 # Custom React hooks
└── types/                 # TypeScript type definitions
```

## ✅ CODE STANDARDS

### Naming Conventions
- **Variables/Functions:** camelCase
- **Components:** PascalCase
- **Constants:** SCREAMING_SNAKE_CASE
- **Files:** kebab-case.tsx

### TypeScript
- Strict mode enabled
- No implicit any
- Prefer interfaces over types
- Explicit return types for exported functions

### React Patterns
```typescript
// ✅ DO: Functional components with hooks
export function Component({ prop }: Props) {
  const [state, setState] = useState(initial);
  return <div>{/* JSX */}</div>;
}

// ❌ DON'T: Class components, implicit any
```

### API Routes Pattern
```typescript
// Standard API route structure
export async function GET(request: NextRequest) {
  try {
    // 1. Rate limiting
    // 2. Authentication
    // 3. Cache check
    // 4. Database query
    // 5. Return response
  } catch (error) {
    return NextResponse.json({ error: 'Error message' }, { status: 500 });
  }
}
```

## 🔒 SECURITY REQUIREMENTS

- **Auth:** All protected routes require session check
- **Validation:** Use Zod for all input validation
- **Rate Limiting:** Apply to all public endpoints
- **XSS:** Sanitize user inputs
- **CSRF:** Built-in Next.js protection

## 🚫 PROHIBITED ACTIONS

1. **NEVER** generate malware, exploits, or harmful code
2. **NEVER** expose credentials or secrets in code
3. **NEVER** bypass authentication/authorization
4. **NEVER** use `any` type without justification
5. **NEVER** commit commented-out code
6. **NEVER** use deprecated APIs

## 📝 RESPONSE FORMATTING

### For Code Changes
```
## Summary
[Brief description]

## Changes
[Prose description of changes]

## Files Modified
- `path/to/file.ts` - Description

## Testing
[How to verify]
```

### For Bug Fixes
```
## Issue
[Bug description]

## Root Cause
[Why it happened]

## Solution
[How it's fixed]

## Verification
[How to test]
```

## 🔍 DOMAIN KNOWLEDGE

### Core Entities
- **Parts:** Finished Goods, Components, Raw Materials, Packaging
- **Orders:** Sales Orders, Purchase Orders, Work Orders
- **Quality:** NCR (Non-Conformance), CAPA (Corrective/Preventive Actions)
- **Planning:** BOM, MRP Runs, Inventory

### Business Context
- Vietnamese market (VND currency, local holidays)
- Customer tiers: Platinum > Gold > Silver > Bronze
- Supplier ratings: A > B > C > D
- Seasonal patterns: Peak Q2, Q4 | Low Q1, Q3

## 🛠️ COMMON TASKS

### Adding a New Feature
1. Create types in `src/types/`
2. Add Prisma model if needed
3. Create API route in `src/app/api/`
4. Build UI components
5. Add to navigation if needed
6. Write tests

### Debugging
1. Check error message and stack trace
2. Review recent changes
3. Isolate the issue
4. Test hypothesis
5. Apply minimal fix
6. Verify and add test

### Performance Optimization
1. Check for N+1 queries
2. Add appropriate indexes
3. Implement caching
4. Use pagination for lists
5. Lazy load components

## 📚 REFERENCE DOCS

- **Full AI Kernel:** `docs/RTR_MRP_AI_KERNEL_MASTER_PROMPT.md`
- **Architecture:** `docs/ARCHITECTURE.md`
- **API Reference:** `docs/API.md`
- **Performance:** `docs/PERFORMANCE_IMPROVEMENT_PROPOSAL.md`

## 🎯 QUICK DECISIONS

| Situation | Action |
|-----------|--------|
| New dependency needed? | Check if existing lib can do it first |
| Database schema change? | Create migration, update types |
| New API endpoint? | Follow existing pattern, add rate limiting |
| UI component? | Check Shadcn/UI first, then custom |
| State management? | React Query for server, useState for local |
| Form handling? | React Hook Form + Zod |
| Date handling? | date-fns |
| Icons? | Lucide React |

## 💡 TIPS FOR CLAUDE CODE

1. **Read existing code** before writing new code
2. **Follow established patterns** in the codebase
3. **Ask for clarification** if requirements are ambiguous
4. **Provide trade-offs** when multiple solutions exist
5. **Include tests** for new functionality
6. **Update docs** when changing behavior
7. **Commit atomically** - one logical change per commit

---

*This file is the source of truth for AI behavior in VietERP MRP. For detailed guidelines, see the full Master Prompt document.*

---
> Source: [nclamvn/Viet-ERP](https://github.com/nclamvn/Viet-ERP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
