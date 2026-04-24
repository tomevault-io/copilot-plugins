## real-estate-crm-project

> Always update memory bank when making changes


# Update Memory Bank on Changes

When you make any change to the codebase (features, fixes, refactors, config), update the memory bank so future sessions have accurate context.

## What to Update

| Change type | Update |
|-------------|--------|
| New feature, fix, or refactor | `activeContext.md` — add to Recent Completed Work, update Next Steps |
| Architecture or pattern change | `systemPatterns.md` |
| Tech stack or constraint change | `techContext.md` |
| Product flow or vision change | `productContext.md` |
| Major milestone | `progress.md` |

## How to Update

1. **After completing work** — Add a brief entry to `activeContext.md` under "Recent Completed Work" (2–4 bullets).
2. **Keep it concise** — One entry per logical change; avoid duplicating code.
3. **Update "Next Steps"** — If priorities shifted, reflect that in activeContext.

## Example

After migrating `apiRequest` to `apiClient`:

```markdown
**apiRequest → apiClient Migration** (Completed - Mar 2025)
- Migrated moderation, billingAdmin, rbacAdmin to apiGet/apiPost/apiPut/apiDelete
- Deprecated apiRequest in queryClient.ts
- Updated ARCHITECTURE.md and REBUILD_PLAN.md
```

## When to Skip

- Trivial edits (typos, formatting only)
- Exploratory changes you revert before finishing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/malhajri07) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
