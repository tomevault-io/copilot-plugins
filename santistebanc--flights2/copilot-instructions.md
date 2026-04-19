## flights2

> Debugging cleanup best practices to ensure minimal, clean fixes


# Debugging Cleanup Best Practices

## Core Principle

**Each debug session should result in the bare minimum changes to fix the issue, with no traces of unsuccessful debugging attempts.**

## Process

### 1. Track Debugging Changes

- Keep a mental log of all debugging changes made during investigation
- Note the purpose and expected outcome of each change
- Document which files/components are being modified

### 2. Identify Root Cause

- Once the actual issue is found, clearly identify what caused it
- Distinguish between debugging attempts and the actual solution
- Verify the fix works without any debugging code

### 3. Clean Up Immediately

- **Remove all debugging changes that didn't contribute to the solution**
- This includes:
  - Temporary console.log statements
  - Conditional rendering guards that weren't needed
  - Unused imports added during debugging
  - Refactoring attempts that didn't solve the issue
  - State management changes that were just attempts

### 4. Commit Minimal Fix

- Only keep changes that actually fix the root cause
- Ensure the fix is the smallest possible change needed
- Test that the solution works without any debugging artifacts

## Examples

### ❌ Bad: Leaving Debugging Traces

```typescript
// Added during debugging but not needed
const isInitialRender = useRef(true);
useEffect(() => {
  const timer = setTimeout(() => {
    isInitialRender.current = false;
  }, 100);
  return () => clearTimeout(timer);
}, []);

// Conditional rendering that wasn't the real fix
{!isInitialRender.current && <Component />}
```

### ✅ Good: Clean Fix

```typescript
// Only the actual fix - memoizing dependencies
const memoizedParams = useMemo(
  () => ({
    departureAirport: departureAirport?._id,
    arrivalAirport: arrivalAirport?._id,
  }),
  [departureAirport?._id, arrivalAirport?._id]
);
```

## When to Apply

- After every debugging session
- Before committing any fixes
- When reviewing code changes
- When the root cause is different from initial assumptions

## Success Criteria

- Code is cleaner than before debugging started
- Only the minimal fix remains
- No debugging artifacts or temporary code
- The solution is well-documented and understood

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/santistebanc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
