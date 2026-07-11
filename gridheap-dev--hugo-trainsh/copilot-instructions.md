## hugo-trainsh

> - Always query context7 for the most recent docs and best practices.

# Instructions for hugo-trainsh

## General Instructions
- Always query context7 for the most recent docs and best practices.
- All comments, logs and documentations in English.
- Include only brief end-user instructions in the root README.md file.
- Place detailed development documentation in docs/*.md (use lowercase filenames).
- Always prioritize ast-grep (cmd: `sg`) over regex/string-replace for code manipulation, using AST patterns to ensure structural accuracy and avoid syntax errors. Examples:
    1. Swap Args: `sg run -p 'fn($A, $B)' -r 'fn($B, $A)'`
    2. Wrap Error: `sg run -p 'return $E' -r 'return wrap($E)'`
    3. API Update: `sg run -p 'user.id' -r 'user.get_id()'`
- No legacy code, no backward compatibility.

---
> Source: [gridheap-dev/hugo-trainsh](https://github.com/gridheap-dev/hugo-trainsh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
