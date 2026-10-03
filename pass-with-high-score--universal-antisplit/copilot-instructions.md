## universal-antisplit

> 1. **File Size Limit**:

# Universal Anti-Split Development Rules

1. **File Size Limit**:
   - Every file MUST NOT exceed **500 lines** (`wc -l < 500`).
   - If a file approaches or exceeds 500 lines, decompose and extract modular sub-components (e.g. into `components/` subdirectories).

2. **String Resources**:
   - DO NOT hardcode user-facing strings or labels directly in Compose code.
   - All strings MUST be declared in `app/src/main/res/values/strings.xml` before being used.
   - In Compose, use `stringResource(R.string.<string_name>)`.
   - In ViewModel / non-composable code, pass string resource IDs or use `context.getString(R.string.<string_name>)`.

---
> Source: [pass-with-high-score/universal-antisplit](https://github.com/pass-with-high-score/universal-antisplit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
