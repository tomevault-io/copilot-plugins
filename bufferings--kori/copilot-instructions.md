## changeset

> Rules for creating changeset files for version management

# Changeset Operation Rules

## Overview

This project uses [Changesets](https://github.com/changesets/changesets) for monorepo version management and publishing. Since the AI assistant cannot interact with CLI prompts, we use a manual changeset file creation approach.

## Workflow

### 1. Making Changes

When code changes are made that warrant a release:

1. User requests: "Create changeset"
2. AI assistant creates changeset file manually

### 2. Changeset File Creation

- AI creates `.changeset/[descriptive-name].md` file
- Uses patch/minor/major based on change type
- Includes clear description in English
- Focus on difference relative to original baseline, not just current working changes

**Example changeset file:**

```markdown
---
'@korix/cors-plugin': patch
---

Fix header configuration bug in CORS plugin
```

## Commands

```bash
# Enter alpha mode (already done)
pnpm changeset pre enter alpha

# Exit alpha mode (for stable release)
pnpm changeset pre exit

# Re-enter alpha mode
pnpm changeset pre enter alpha
```

---
> Source: [bufferings/kori](https://github.com/bufferings/kori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
