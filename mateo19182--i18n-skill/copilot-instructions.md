## i18n-skill

> Handle internationalization (i18n) using i18next-cli - detect hardcoded strings, extract translation keys, validate translations


## Overview

This skill provides a complete workflow for managing translations using **i18next-cli**.

The project uses:
- **i18next** / **react-i18next** for translations
- **i18next-cli** for key extraction and linting
- **Locale files** in `src/locales/{lang}/translation.json`

## Quick Workflow

### Option 1: Automated Replacement (Recommended)

```bash
pnpm i18n:translate
```

This script:
- Parses lint output to find hardcoded string locations
- Maps strings to translation keys from primary locale
- Replaces them with `t("key")` calls
- Adds `useTranslation` import and hook if missing

**Note**: If strings have no translation keys, the script will skip them. Add keys first.

### Option 2: Manual Replacement

For manual replacement:

1. Find the string in the component file
2. Replace hardcoded text with `t("key")`
3. Add `import { useTranslation } from "react-i18next"` if missing
4. Add `const { t } = useTranslation()` inside the component function

Example:
```tsx
// Before
<p>Cargando...</p>

// After
import { useTranslation } from "react-i18next";

export function MyComponent() {
  const { t } = useTranslation();
  // ...
  <p>{t("common.loading")}</p>
}
```

## Step-by-Step Process

### 1. Find Hardcoded Strings
```bash
pnpm i18n:lint
```

### 2. Add Translation Keys
```bash
pnpm i18n:add "category.key" "English value"
```

### 3. Run Automated Replacement
```bash
pnpm i18n:translate
```

### 4. Verify
```bash
pnpm typecheck
pnpm i18n:lint
```

## Key Naming Conventions

- **Structure**: `category.subcategory.key` (e.g., `common.loading`)
- **Categories**: Use existing or create new (e.g., `common`, `students`)
- **Group related keys** under a category namespace

## Common Errors

- **Cannot find name 't'** - Add `const { t } = useTranslation()` to component
- **'useTranslation' is declared but never used** - You need to actually use `t()` in the component

---
> Source: [mateo19182/i18n-skill](https://github.com/mateo19182/i18n-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
