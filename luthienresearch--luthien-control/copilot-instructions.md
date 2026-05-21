## fail-hard

> In general, code should NOT 'fallback' to another solution when something doesn't work as expected *unless specifically told otherwise*. Code should default to failing quickly, loudly, and obviously.

In general, code should NOT 'fallback' to another solution when something doesn't work as expected *unless specifically told otherwise*. Code should default to failing quickly, loudly, and obviously.

Prefer raising exceptions over not doing so. Avoid Union[...] and Optional[T] types as both arguments and return values. Branching conditional logic should correspond to business logic distinctions, not trying to recover from ambigious input values.

---
> Source: [LuthienResearch/luthien_control](https://github.com/LuthienResearch/luthien_control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
