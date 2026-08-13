## anti-slop

> - `src/` is the canonical plugin implementation.

# Repository guidance

- `src/` is the canonical plugin implementation.
- Keep rules generic and suitable for reuse across repositories. Do not add application-specific names, paths, or exceptions.
- Use Oxlint's ESTree API; do not add another production parser.
- Add focused RuleTester coverage for semantic rule changes.
- Run `pnpm sync:skill-assets` after changing production source.
- Run `pnpm check` before committing.

---
> Source: [dmmulroy/anti-slop](https://github.com/dmmulroy/anti-slop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
