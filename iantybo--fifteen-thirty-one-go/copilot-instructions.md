## fifteen-thirty-one-go

> This repo uses Cursor rules in `.cursor/rules/*.mdc`.

This repo uses Cursor rules in `.cursor/rules/*.mdc`.

Key workflow rule:
- After every code generation/edit cycle that changes code, run:

```bash
coderabbit review --plain --no-color --type all --base main
```

and iterate on findings until there are no findings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/iantybo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
