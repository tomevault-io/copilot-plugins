## gemeenteportaal

> - `X-ALT-DESC;FMTTYPE=text/html` intentionally contains HTML markup such as

# Repository guidance

## ICS HTML compatibility

- `X-ALT-DESC;FMTTYPE=text/html` intentionally contains HTML markup such as
  `<strong>`, `<br />`, and `<a>`. Church App relies on this markup to render
  calendar descriptions correctly.
- Do not HTML-escape the complete HTML description or otherwise turn its
  structural tags into text entities.
- If dynamic values need additional hardening, escape only those individual
  text values before interpolation while preserving the surrounding markup.
- Treat changes to ICS HTML generation as a Church App compatibility concern
  and verify the rendered result in Church App before merging.
- Unescaped interpolation of dynamic ICS values predates the duty-elder
  feature; it is not by itself a regression introduced by that feature.

---
> Source: [JensdeVlaming/gemeenteportaal](https://github.com/JensdeVlaming/gemeenteportaal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
