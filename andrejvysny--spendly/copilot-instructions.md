## import-wizard

> When working on the Import wizard (upload, configure, map, clean, process steps).


# Import wizard

- Prefer delegating to the `import-wizard` subagent for import wizard work.
- Steps: upload → configure → map → clean → confirm/process. Backend: ImportWizardController (routes under imports/wizard).
- Key services: TransactionImportService, CsvProcessor, TransactionRowProcessor; mapping and persistence in app/Services/TransactionImport/, app/Services/Csv/.
- When changing steps or mappings: keep backend step APIs and frontend wizard state in sync. Frontend: resources/js/pages/import/ (ImportWizard, wizard-steps/).
- Reference: app/Http/Controllers/Import/ImportWizardController.php, resources/js/pages/import/components/ImportWizard.tsx.
- **CLI import**: CSV can be imported from the terminal via `php artisan import:csv <file> --account=...`. See AGENTS.md (section "CLI and AI agent usage") for options and examples.

---
> Source: [andrejvysny/spendly](https://github.com/andrejvysny/spendly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
