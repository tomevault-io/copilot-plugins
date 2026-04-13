## wishlist

> - **All documentation files MUST be placed in the `/docs` folder**

# Documentation Rules

## Documentation Location

- **All documentation files MUST be placed in the `/docs` folder**
- Never create markdown documentation files in the project root
- Keep the root clean with only essential files (README.md, package.json, etc.)

## When to Create Documentation

Documentation should be created and kept in source control ONLY if it meets these criteria:

### ✅ CREATE DOCUMENTATION FOR:

- **Complex Integrations** - Thirdweb, Supabase, external APIs
- **Architecture Decisions** - Why certain approaches were chosen
- **Setup Guides** - Required environment variables, configuration steps
- **User Guides** - How end-users should use major features
- **Cross-Cutting Concerns** - Authentication, caching, deployment
- **Known Gotchas** - Issues that future developers will encounter (e.g., smart account auth)

### ❌ DO NOT CREATE DOCUMENTATION FOR:

- **Simple Bug Fixes** - These are documented in commit messages
- **Feature Implementation Details** - Code should be self-documenting
- **Debugging Sessions** - Temporary troubleshooting notes
- **Implementation Changelogs** - Use git history instead
- **Duplicate Information** - If it exists elsewhere, link to it
- **Prototype Summaries** - Once feature is complete, document the final version only

## Documentation Format

When creating documentation:

1. **Use a clear, descriptive filename** - e.g., `FEATURE_NAME_GUIDE.md`
2. **Include these sections:**
   - Overview - What is this?
   - Setup/Prerequisites - What's needed?
   - Usage - How to use it?
   - Technical Details - How does it work?
   - Troubleshooting - Common issues
3. **Update `/docs/README.md`** - Add your new doc to the index
4. **Keep it current** - Update docs when implementation changes
5. **Avoid duplication** - Reference existing docs instead of repeating

## Documentation Maintenance

### When to Delete Documentation:

- Feature has been removed
- Information is outdated and no longer relevant
- Content has been consolidated into another doc
- Temporary debugging guides after issue is resolved

### When to Update Documentation:

- Implementation has changed
- New configuration options added
- Known issues have been resolved
- User workflows have changed

## Example Good Documentation Structure:

```
/docs
  README.md                      # Index of all docs
  THIRDWEB_API.md               # Integration guide
  GIFT_EXCHANGE_SETUP.md        # Complex feature setup
  SIWE_AUTHENTICATION.md        # Cross-cutting concern
  USER_GUIDE_EXCHANGES.md       # End-user documentation
```

## Example Bad Documentation Structure:

```
/docs
  BUG_FIX_PURCHASER_123.md      # ❌ Bug fixes don't need docs
  IMPLEMENTATION_NOTES_V1.md    # ❌ Use git history
  DEBUGGING_SESSION_NOV_2.md    # ❌ Temporary troubleshooting
  COMPONENT_DETAILS.md          # ❌ Code should be self-documenting
  CHANGELOG_FEATURE_X.md        # ❌ Use git commits
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mykcryptodev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
