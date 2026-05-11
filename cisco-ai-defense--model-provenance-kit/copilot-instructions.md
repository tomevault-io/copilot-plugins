## no-cursor-co-author

> Never add Cursor as a co-author in git commits


# No Cursor Co-Author in Commits

When creating git commits, NEVER add a `Co-authored-by` trailer for Cursor, Cursor Agent, or any AI assistant.

Commits must only attribute human authors. Do not append lines like:

```
Co-authored-by: Cursor <cursor@cursor.com>
Co-authored-by: cursor[bot] <...>
```

Keep commit messages clean with only the summary and description. No AI attribution trailers.

---
> Source: [cisco-ai-defense/model-provenance-kit](https://github.com/cisco-ai-defense/model-provenance-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
