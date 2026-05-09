## linter

> Linter Guidelines


# Linter

We have a built-in ESLint linter configured at the root folder. After making any significant change, always run the linter with `pnpm lint` and fix all potential issues.

## Most common mistakes that lead to linter errors

### Lint Rule: Unexpected nullable string value in conditional. Please handle nullish or empty cases explicitly

To prevent this, when comparing strings, instead of writing:

```
if (!str) {}
```

do this:

```
if (str !== undefined) {}
```

---
> Source: [caido-community/authswap](https://github.com/caido-community/authswap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
