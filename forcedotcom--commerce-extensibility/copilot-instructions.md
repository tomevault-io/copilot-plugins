## commerce-extensibility

> Sample implementations for extending Salesforce Commerce services (pricing, buyer groups, tax, shipping, checkout, etc.).

# Commerce Extensibility Repository

Sample implementations for extending Salesforce Commerce services (pricing, buyer groups, tax, shipping, checkout, etc.).

## Repository Structure

```
commerce/
├── domain/      # Business logic extensions (pricing, tax, buyer groups, etc.)
└── endpoint/    # API extensions (cart, search, account)
```

---

## Apex v67.0 Migration

Reference: [Apex Release Notes - Summer '26](https://help.salesforce.com/s/articleView?id=release-notes.rn_apex.htm&release=262&type=5)

When a user asks to migrate files for Apex v67.0, follow this process:

#### Step 1: Analyze

Scan `.cls` files and identify issues based on the v67.0 Secure by Default changes:

**Sharing Keywords** - Classes without a sharing keyword now default to `WITH SHARING`:
```apex
// Explicitly declare the sharing mode you need
public without sharing class MyExtension { }
public with sharing class MyExtension { }
public inherited sharing class MyExtension { }
```

**Query Mode** - Queries now run as `USER_MODE` by default:
```apex
// Add WITH SYSTEM_MODE only if you need to bypass user permissions
[SELECT Id FROM Account WHERE Id = :id WITH SYSTEM_MODE]
```

**DML Access Level** - DML operations now run as `USER_MODE` by default:
```apex
// Add SYSTEM_MODE only if you need to bypass user permissions
Database.insert(records, AccessLevel.SYSTEM_MODE);
insert as system records;
```

**Deprecated Syntax** - `WITH SECURITY_ENFORCED` is deprecated:
```apex
// ❌ Deprecated
[SELECT Name FROM Account WITH SECURITY_ENFORCED]

// ✅ Use WITH USER_MODE
[SELECT Name FROM Account WITH USER_MODE]
```

#### Step 2: Show Diff to User

Present each change clearly with file name, line numbers, what will change, and why.

If user asks, show detailed before/after diff for each change.

#### Step 3: Get Approval

```
Should I apply these X changes across Y files? (yes/no)

Reply 'yes' to proceed, or ask me to show details for any file.
```

**Wait for explicit "yes" before proceeding.**

#### Step 4: Apply Changes

1. Update code with required changes
2. Keep API version unchanged (backward compatibility)
3. Preserve existing formatting

#### Step 5: Report

```
✅ Migration complete!

Updated:
- File1.cls (4 queries, docs)
- File2.cls (docs only)

Next steps:
1. Review changes
2. Run tests
3. Commit when ready
```

---
> Source: [forcedotcom/commerce-extensibility](https://github.com/forcedotcom/commerce-extensibility) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
