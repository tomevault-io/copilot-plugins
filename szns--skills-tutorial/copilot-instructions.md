## skills-tutorial

> - **Framework:** Next.js (App Router preferred)

## Tech Stack Context
- **Framework:** Next.js (App Router preferred)
- **Language:** TypeScript (Strict mode)
- **Styling:** Tailwind CSS
- **Icons:** Lucide-React

## Architectural Standards

### 1. Dependency Management (Native-First)
- **Rule:** Favor native Web APIs over adding new NPM packages.
- **Guideline:** Use `fetch()` for networking and `Intl` for dates, times, and currencies. 
- **Constraint:** Do not introduce libraries like `axios`, `moment`, or `dayjs` unless they already exist in the `package.json`.

### 2. Analytics & Tracking
- **Rule:** Use the internal abstraction layer for all event tracking.
- **Pattern:** `analytics.track(eventName, properties)`
- **Constraint:** Never call `segment.track`, `window.gtag`, or `fbq` directly. All tracking must be routed through our utility to ensure schema consistency.

### 3. Error Handling (The Result Pattern)
- **Rule:** Use a Tuple-based "Result" pattern for asynchronous operations.
- **Pattern:** `const [data, error] = await asyncFunction();`
- **Constraint:** Avoid `try/catch` blocks for flow control. Functions should return `[T, Error | null]` rather than throwing exceptions. This ensures predictable logic branches.

### 4. UI & Styling
- **Rule:** All styling must be handled via Tailwind CSS utility classes.
- **Constraint:** No inline `style={{ ... }}` objects. No CSS-in-JS. Keep the UI consistent with the design system tokens.

## Code Example
Follow this pattern:

```typescript
// Standard: Result Pattern + Native Fetch + Internal Analytics
export const fetchUserData = async (userId: string) => {
  const [res, err] = await safeFetch(`/api/users/${userId}`);
  
  if (err) {
    analytics.track('user_fetch_failed', { userId, error: err.message });
    return [null, err];
  }

  return [res, null];
};

---
> Source: [SZNS/skills-tutorial](https://github.com/SZNS/skills-tutorial) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
