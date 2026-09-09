## avalonia-templates

> Before starting any task in this repository, read [`DEVELOPMENT.md`](DEVELOPMENT.md).

# Repository guidance

Before starting any task in this repository, read [`DEVELOPMENT.md`](DEVELOPMENT.md).
It explains how the templates are structured and how to build, test, and extend them.

Key conventions:
- Templates live under `templates/csharp/` and `templates/fsharp/`, described by a `.template.config/` folder.
- A parameter must be added in all three files: `template.json` (symbol), `dotnetcli.host.json` (CLI), and `ide.host.json` (IDE).
- Add build coverage for new templates or parameters by adding a `New-Case` row to the `$builds` array in `tests/build-test.ps1` (item templates go in `$itemTemplates`), and document user-facing changes in `README.md`.

---
> Source: [AvaloniaUI/Avalonia.Templates](https://github.com/AvaloniaUI/Avalonia.Templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
