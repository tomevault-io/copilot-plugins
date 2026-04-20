## cursor-ai-procedures-example

> CRITICAL rule to prevent breaking API contracts without explicit instruction. Do not change field names, request/response structures, or API behavior without being explicitly told to change the contract.


# API Contract Stability Rule

## ⚠️ CRITICAL: Do Not Change API Contracts Without Explicit Instruction

**NEVER modify API contracts (field names, request/response structures, endpoint behavior) unless the user explicitly asks you to change the contract.**

## What This Means

### ❌ DO NOT:
- Change field names in DTOs to "fix" what appears to be a mismatch with frontend
- Add aliases or alternative field names "for convenience"
- Modify request/response structures to accommodate what seems like frontend bugs
- Change API behavior to work around frontend implementation issues
- Assume the frontend is wrong and "fix" it in the backend

### ✅ DO:
- Report mismatches between frontend and backend API contracts
- Ask the user which side should be fixed (frontend or backend)
- Document API contract issues for the user to decide
- Only change contracts when explicitly instructed

## Examples of What NOT To Do

### Bad: Adding Field Aliases
```typescript
// ❌ DON'T do this without being asked
publicUrl?: string;
profileImageUrl?: string; // "Alias for convenience"
```

### Bad: Changing Field Names
```typescript
// ❌ DON'T rename fields to match frontend
// Original: publicUrl
// Changed to: profileImageUrl (because frontend uses it)
```

### Bad: Modifying Request Structure
```typescript
// ❌ DON'T change request structure to accommodate frontend
// If frontend sends wrong field name, report it, don't add support for it
```

## What To Do Instead

### ✅ Correct Approach
1. **Identify the mismatch** between frontend and backend
2. **Report to user** what the frontend is sending vs what the API expects
3. **Ask which should be fixed** - frontend code or backend contract
4. **Wait for explicit instruction** before making changes
5. **Document the issue** if it's a bug that needs fixing

## Why This Matters

- **API contracts are intentional** - field names and structures are designed decisions
- **Frontend/backend mismatches** are bugs that need to be fixed on the correct side
- **Adding aliases/alternatives** creates technical debt and confusion
- **Breaking changes** require coordination and documentation
- **Consistency** is important for maintainability

## Enforcement

This rule is marked `alwaysApply: true` - it must be followed for ALL API-related changes, no exceptions.

When in doubt: **Ask the user which side should be fixed, don't assume.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/avermhir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
