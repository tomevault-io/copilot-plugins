## toml-string-safety

> TOML string pre-processing safety rules for fixTomlArrayTables


# TOML String Pre-processing

`fixTomlArrayTables` pre-processes LLM-generated TOML before parsing.

## Critical: Preserve triple-quoted strings

TOML `"""..."""` multi-line strings must NOT be matched by single-line quote fixers.
Always use negative lookahead `(?!"{3})` before matching `"(.*)"` patterns.

```typescript
// BAD — mangles """ into "\""
/^(\s*\w[\w.-]*\s*=\s*)"(.*)"$/gm

// GOOD — skips triple-quoted lines
/^(\s*\w[\w.-]*\s*=\s*)(?!"{3})"(.*)"$/gm
```

## Common LLM TOML mistakes to handle

1. Unescaped inner quotes: `description = "识别为"值得沉淀"的标准"`
2. Misformed multi-line strings: `system_prompt = "\""` followed by bare text
3. Missing `[[array]]` table headers turning into invalid keys

---
> Source: [zhangdszq/teamclaw](https://github.com/zhangdszq/teamclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
