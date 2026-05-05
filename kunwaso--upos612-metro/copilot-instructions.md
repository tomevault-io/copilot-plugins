## migration-before-ui

> Ensure migration exists for every persisted form/settings field — no UI-first without DB columns


# Migration Before UI — Form & Settings

When adding or changing **form fields or settings** that are saved to the database:

1. **Do not** add the field to the Blade form, controller, or Util until a migration exists that adds or alters the corresponding column(s).
2. **Order:** Implement migration first (or in the same change set), then model, Form Request, Util, controller, Blade.
3. **Cross-check:** Every key in the Form Request `rules()` that is persisted to the model/table must have a column in the migration. Exclude file inputs, `_token`, and UI-only flags like `avatar_remove`.
4. **Run:** Ensure the migration has been run (`php artisan migrate` or `php artisan module:migrate ModuleName`) before calling the feature done.

See **AGENTS.md** Section 4 (build order), Section 4.1 Step 1, and Section 9 checklist (migration for every new persisted field).

---
> Source: [kunwaso/upos612-metro](https://github.com/kunwaso/upos612-metro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
