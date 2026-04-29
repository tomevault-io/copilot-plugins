## weddingflo

> - ✅ ALWAYS use `sessionClaims.metadata.role` for role checks

# WeddingFlo - Cursor AI Rules
# Auto-loaded on every Cursor session
# Last Updated: October 23, 2025

## 🎯 MANDATORY RULES - ENFORCE ON EVERY CODE CHANGE

### 1. SESSION CLAIMS FOR AUTH (NO DATABASE)
- ✅ ALWAYS use `sessionClaims.metadata.role` for role checks
- ✅ ALWAYS use `sessionClaims.metadata.company_id` for company ID
- ❌ NEVER query database for role, company_id, or auth data
- ❌ NEVER add database queries to middleware for auth
- ❌ NEVER add database queries to layouts for auth
- Target: <5ms auth performance (session claims only)

### 2. OCTOBER 2025 SUPABASE API STANDARDS
- ✅ ALWAYS use `@supabase/supabase-js` package
- ✅ ALWAYS use `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` env var
- ✅ ALWAYS use `SUPABASE_SECRET_KEY` for admin operations
- ❌ NEVER use `@supabase/ssr` package
- ❌ NEVER use `SUPABASE_ANON_KEY` (deprecated)
- ❌ NEVER use JWT templates (use default Clerk JWT)
- ❌ NEVER make `createServerSupabaseClient()` async

### 3. MINIMAL MIDDLEWARE PATTERN
- ✅ ONLY JWT verification in middleware
- ❌ NO database queries in middleware
- ❌ NO role checks in middleware (do in layouts)
- ❌ NO Supabase client creation in middleware
- Target: <1ms middleware execution

### 4. PROFESSIONAL STANDARDS
- ✅ NEVER use `any` type (use `unknown` if needed)
- ✅ ALWAYS use Zod for input validation
- ✅ ALWAYS use comprehensive error handling (TRPCError)
- ✅ ALWAYS use generated database types
- ✅ ALWAYS read migration files before database code
- ❌ NO band-aid code or "TODO: fix later"
- ❌ NO assumptions about database schema

### 5. DUAL SYNC PATTERN (WEBHOOKS)
- ✅ ALWAYS update BOTH Supabase AND Clerk metadata
- ❌ NEVER update only Supabase (session claims won't work)
- ❌ NEVER update only Clerk (database out of sync)

### 6. INCREMENTAL TESTING
- ✅ Change ONE file → test → next file
- ✅ Run `npm run build` after each change
- ❌ NEVER batch 10+ changes and test at the end

## 🚨 RED FLAGS - REJECT CODE IMMEDIATELY

If you see ANY of these, STOP and warn user:
- Database queries in middleware
- Database queries for role/company_id in layouts
- `SUPABASE_ANON_KEY` environment variable
- `@supabase/ssr` package import
- `any` types without justification
- Missing error handling
- `localStorage` for auth tokens
- Manual JWT decoding
- JWT templates (no template exists by default)
- Making `createServerSupabaseClient` async
- Only updating Supabase OR Clerk (must update BOTH)

## 📚 REFERENCE FILES - READ BEFORE CODE CHANGES

Before modifying auth, database, or core architecture:
1. `.claude/WEDDINGFLOW_PERMANENT_STANDARDS.md` - Complete standards
2. `docs/FINAL_ARCHITECTURE_AND_DEPLOYMENT_STRATEGY.md` - Architecture
3. `docs/SUPABASE_CLERK_WORKING_MODEL.md` - Supabase + Clerk integration

Use `/standards` command to load permanent standards.

## 🎯 DECISION MATRIX

Before modifying working code:
1. Is it broken? NO → Ask: Does user request this change?
2. Does user request it? NO → Ask: Is there a concrete problem?
3. Concrete problem? NO → LEAVE IT ALONE (if it works, don't touch)
4. If YES to any → Ask user WHY it's needed before proceeding

## ✅ CORRECT PATTERNS (ALWAYS USE THESE)

### Session Claims:
```typescript
const { userId, sessionClaims } = await auth()
const role = sessionClaims?.metadata?.role  // ✅ Fast, <5ms
const companyId = sessionClaims?.metadata?.company_id  // ✅ No DB query
```

### Supabase Client:
```typescript
import { createClient } from '@supabase/supabase-js'  // ✅ NOT @supabase/ssr

const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY!  // ✅ NOT anon
)
```

### Middleware:
```typescript
export default clerkMiddleware((auth, req) => {
  if (!isPublicRoute(req)) {
    auth().protect()  // ✅ ONLY JWT verification
  }
  // ✅ NO database queries
  // ✅ NO role checking
})
```

### tRPC Error Handling:
```typescript
try {
  // Operation...
} catch (error) {
  if (error instanceof TRPCError) throw error

  throw new TRPCError({
    code: 'INTERNAL_SERVER_ERROR',
    message: 'User-friendly message',
    cause: error,
  })
}
```

## 🎬 SESSION START CHECKLIST

At start of EVERY session:
1. Verify auth works (sign in → dashboard loads)
2. Run `npm run build` (must show "Compiled successfully")
3. Check git status (should be clean)

If ANY fail → fix FIRST before new work.

## 🎓 KEY LESSONS FROM HISTORY

Session 19 Disaster (Oct 18, 2025):
- Changed to JWT templates → broke auth
- Switched to `@supabase/ssr` → wrong package
- Made client function async → broke 50+ files
- Batch changed multiple files → hard to debug
- **Result:** Lost 2-3 days of work

**Lesson:** If it works, don't touch it. One change at a time. Test incrementally.

## 📊 PERFORMANCE TARGETS

- Middleware: <1ms (JWT only)
- Layout auth: <5ms (session claims)
- tRPC context: <5ms (session claims)
- Total auth overhead: <10ms
- Database auth (FORBIDDEN): 50-100ms (10-20x slower)

## ✅ COMMITMENT

As Claude Code working on WeddingFlo:
- I will ALWAYS read these rules before suggesting code
- I will REJECT code that violates these patterns
- I will WARN user if their request would break standards
- I will ENFORCE October 2025 API standards
- I will NEVER compromise on production-grade quality

---
**Auto-loaded by Cursor on every session**
**Version:** 2.0 (October 23, 2025)

---
> Source: [nitinthakurcode/weddingflo](https://github.com/nitinthakurcode/weddingflo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
