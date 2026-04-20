## sdscc-manager

> SAFETY / STABILITY RULES


SAFETY / STABILITY RULES
==================================================

Do not change database constraints or indexes — schema changes are only allowed if strictly required for date-based filtering.

Do not introduce new dependencies — avoid adding new libraries or frameworks.

Do not refactor unrelated code — changes must only touch year-as-state behavior and reporting.

Preserve backward compatibility — any existing integrations or APIs that rely on fiscal year logic must not break.

Unit tests / automated tests — if tests touch year logic, update them to use date filtering instead of current year flags.

Code comments — mark all removed/deprecated logic with clear "DEPRECATED: Year-as-state" tags.

No hard-coded dates — all filtering must use record timestamps or existing date fields.

No assumptions about fiscal year boundaries — do not introduce implicit start/end dates except for reporting purposes.

Minimal UI changes — only disable or hide buttons; do not redesign pages or flows.

Rollbacks must be possible — all changes should be easily reversible in case of errors.

DOCUMENTATION RULES
==================================================

Clearly document any disabled logic in rule.md or a separate changelog.

Include a short summary in each affected module explaining how reports now work with date filtering.

Specify what reporting behavior was preserved vs deprecated.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Ricas2410) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
