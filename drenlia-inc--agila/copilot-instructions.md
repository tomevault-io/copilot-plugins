## dirty-save-forms

> Dirty-save forms via JSON snapshot (Save disabled until changes)


# Dirty-save forms

When building editable forms with an explicit Save action, use the JSON snapshot pattern (see also personal skill `dirty-save-forms`):

1. Keep live form state in `settings` (or equivalent).
2. After load (and after a successful save), store a JSON snapshot: `savedSnapshot = JSON.stringify(mapped)`.
3. Dirty check:

```ts
const isDirty = savedSnapshot !== '' && snapshotOf(settings) !== savedSnapshot;
```

4. Save button: `disabled={saving || !isDirty}`.
5. Save handler no-ops if `!isDirty || saving`.
6. On success, refresh the snapshot so the form is clean again.

Any field change via `setField` / `handleToggle` updates `settings` → snapshot differs → Save enables.

---
> Source: [drenlia-inc/agila](https://github.com/drenlia-inc/agila) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-15 -->
