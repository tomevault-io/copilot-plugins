## trustee

> - Use English for code comments.

# Cursor Rules

- Use English for code comments.
- When wrapping errors, preserve and surface the original error details.
  Prefer patterns like `map_err(|e| anyhow!("context: {e}"))` over
  `map_err(|_| anyhow!("context"))`.

---
> Source: [openanolis/trustee](https://github.com/openanolis/trustee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
