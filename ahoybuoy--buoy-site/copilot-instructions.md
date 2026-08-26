## buoy-site

> This file provides guidance to AI tools working with this codebase.

# Project Instructions

This file provides guidance to AI tools working with this codebase.

## Design System Rules

This project uses the Subsequent Saturn Design System. Follow these rules when generating code:

### Anti-Patterns

AVOID:
- `<div onClick>` - Use `<Button>` or semantic elements
- Inline styles for colors/spacing - Use tokens or classes
- Creating component variants that already exist
- Arbitrary values (e.g., `p-[13px]`) - Use scale values

### Validation

Run before committing:
```bash
buoy check          # Quick validation
buoy drift check    # Detailed drift analysis
buoy fix --dry-run  # See suggested fixes
```

---
> Source: [ahoybuoy/buoy-site](https://github.com/ahoybuoy/buoy-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
