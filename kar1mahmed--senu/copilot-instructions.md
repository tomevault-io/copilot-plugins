## senu

> dd strategic console.log messages for debugging with engaging humor, but follow these guidelines:


dd strategic console.log messages for debugging with engaging humor, but follow these guidelines:

**Placement Strategy:**
- API route handlers: Log request start/end with method and endpoint
- Error boundaries and catch blocks: Always log the error with context
- State changes: Log before/after critical state updates
- Authentication flows: Log auth steps (obfuscate tokens)
- External API calls: Log request/response status
- Form submissions: Log validation results

**Data Handling:**
- Long strings: Use `.substring(0, 200) + '...'` for data over 200 chars
- Sensitive data: Obfuscate with `token: '***' + token.slice(-4)` pattern
- Objects: Use `JSON.stringify(obj, null, 2).substring(0, 300)`
- Arrays: Log length and first item: `users: [${users.length} items, first: ${users[0]?.name}]`

**Humor Examples:**
- `console.log('🚀 API call launching faster than my morning coffee:', endpoint)`
- `console.log('🕵️ Auth check: user is', user ? 'legit' : 'sus')`
- `console.log('💾 Database query took', time, 'ms (still faster than loading TikTok)')`

**Avoid:** Login pages, payment processing, production builds, tight loops.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kar1mAhmed) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
