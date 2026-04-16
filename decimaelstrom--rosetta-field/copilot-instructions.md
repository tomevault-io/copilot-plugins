## rosetta-field

> - **NEVER use emojis, Unicode symbols, or non-ASCII characters in .py files**

# Cursor Rules for Rosetta-Field



## Code Style



### Python Files - ASCII Only

- **NEVER use emojis, Unicode symbols, or non-ASCII characters in .py files**

- Use only ASCII text (printable characters 32-126) in Python source code

- This prevents encoding issues across different systems and terminals

- Use plain text descriptions instead of emojis (e.g., "Demo complete" not "✨ Demo complete ✨")

- Use ASCII arrows (->) instead of Unicode arrows (→)

- Use ASCII quotes and apostrophes, not Unicode smart quotes



### Rationale

Windows terminals often use cp1252 encoding which cannot display many Unicode characters, causing UnicodeEncodeError exceptions. Keeping Python files ASCII-only ensures maximum compatibility.



### Exceptions

- Markdown files (.md) may use Unicode/emojis for documentation

- Comments in code should still be readable and clear, just ASCII-only




---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Decimaelstrom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
