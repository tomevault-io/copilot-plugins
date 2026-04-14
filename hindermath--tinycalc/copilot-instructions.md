## tinycalc

> MicroCalc .NET 10 is a modern C# port of the classic Borland Turbo Pascal MicroCalc spreadsheet application. It serves as a showcase for "agentic" porting, demonstrating how legacy code can be systematically transformed into a modern architecture.

# GEMINI.md - MicroCalc .NET 10

## Project Overview
MicroCalc .NET 10 is a modern C# port of the classic Borland Turbo Pascal MicroCalc spreadsheet application. It serves as a showcase for "agentic" porting, demonstrating how legacy code can be systematically transformed into a modern architecture.

- **Primary Language:** C# (.NET 10)
- **UI Framework:** [Terminal.Gui](https://github.com/gui-cs/Terminal.Gui)
- **Architecture:** Clean separation between Domain Logic (Core) and Presentation (TUI).
- **Legacy Context:** Based on the original `CALC.PAS`, `CALC.INC`, and `CALC.HLP` files.

## Project Structure
- `src/MicroCalc.Core/`: Core engine, spreadsheet model, formula parser (recursive descent), and evaluation logic.
- `src/MicroCalc.Tui/`: Terminal.Gui application, including grid rendering, command palette, and help system.
- `tests/MicroCalc.Core.Tests/`: Unit tests for engine calculations, persistence, and formatting.
- `tests/MicroCalc.Tui.Tests/`: Smoke tests for the TUI, including a dedicated `--smoke` mode.
- `docs/`: Documentation, migration plans, and help content.

## Key Commands

### Building the Project
```bash
dotnet restore MicroCalc.sln
dotnet build MicroCalc.sln --configuration Release --no-restore
```

### Running the Application
```bash
dotnet run --project src/MicroCalc.Tui/MicroCalc.Tui.csproj
```

### Running Tests
```bash
dotnet test MicroCalc.sln --configuration Release --no-build
```

### Running Smoke Tests
Executes a non-interactive validation of the TUI components.
```bash
dotnet run --no-build --configuration Release --project src/MicroCalc.Tui/MicroCalc.Tui.csproj -- --smoke
```

## Branching & PR Workflow
- Create working branches either as `codex/<short-topic>` or, for Spec-Kit-driven work, as `NNN-short-description`.
- Do not push directly to `main`; merge via pull request.
- When a dedicated feature branch has implemented the requirements of a Lastenheft, rename that file to `Lastenheft_<Thema>.<feature-branch>.md` so the delivered requirement scope stays traceable in the repository.
- `Directory.Build.props` defines the repo-wide `Version`, `AssemblyVersion`, and `FileVersion` values with `Major.Minor.Patch.Build`; on numbered Spec-Kit branches, `Minor` is the numerically interpreted feature/branch number (`002` -> `2`), `Patch` is the branch commit count after the current commit, and `Build` only increases before `dotnet build` or `dotnet test`.

## Development Conventions

### Architecture & Design
- **Separation of Concerns:** Keep the `MicroCalc.Core` independent of `Terminal.Gui`. The Core should handle all data manipulation and calculations.
- **Formula Engine:** Follow the recursive descent pattern for parsing. Ensure any new functions or operators are added to the evaluator logic and backed by golden tests.
- **Persistence:** Use JSON (`.mcalc.json`) as the primary storage format.

### UI & Interaction
- **Keybindings:** Maintain compatibility with classic navigation (Arrow keys) and the `/` command palette trigger.
- **Help System:** Help content is loaded from `CALC.HLP`. Ensure the `HelpDocument` logic remains robust to file path resolutions.

### Testing
- **Golden Tests:** Use `FormulaGoldenTests.cs` for verifying complex formula calculations against expected outputs.
- **Smoke Tests:** Any major UI refactoring should be verified via the `--smoke` flag.
- **Coverage:** Aim for high coverage in the `Core` project, especially for the formula evaluator.

### Documentation & Language
- Provide documentation and didactic comments bilingually: German first, English second.
- Keep both language blocks at CEFR B2 readability.
- Large normative documents such as `Pflichtenheft*.md` and `Lastenheft*.md` may use a synchronized English sidecar with suffix `.EN.md` instead of an oversized inline-bilingual file; the German version remains canonical unless explicitly marked otherwise.
- Maintain complete XML documentation for affected public APIs (`<summary>`, `<param>`,
  `<returns>`, `<exception>` where applicable).
- Do not suppress CS1591 globally.
- If API signatures or XML comments change, regenerate DocFX output in the same change set.

## Legacy Reference
The original Pascal files (`CALC.PAS`, `CALC.INC`) and the help file (`CALC.HLP`) are stored in the root directory. These are the source of truth for behavior and feature parity.

## Documentation
- `PLAN_MICROCALC_CSHARP_DOTNET10.md`: Detailed migration strategy and feature mapping.
- `CONTRIBUTING.md`: Guidelines for contributions.
- `README.md`: General project overview and quick start.

## Project Statistics

- When shared AI-agent guidance, workflow conventions, or statistics methodology changes, review and update `AGENTS.md`, `CLAUDE.md`, `GEMINI.md`, and `.github/copilot-instructions.md` together when they are affected.
- Shared guidance must not be updated in only one of these files; any intentional agent-specific divergence must be documented in the same change.
- Maintain `docs/project-statistics.md` as the living statistics ledger for the repository.
- Update the file after each completed Spec-Kit implementation phase, after each agent-driven repository change, or when a refresh is explicitly requested.
- Within the `## Fortschreibungsprotokoll` table, keep entries in strict chronological order: oldest entry at the top, newest and most recently added entry at the bottom; entries with the same date keep their insertion order.
- Keep a final top-level `## Gesamtstatistik` block as the last section of `docs/project-statistics.md`; do not append later top-level sections after it.
- Inside that final `## Gesamtstatistik` block, maintain compact ASCII-only trend diagrams directly below the textual overall summary and refresh them with every statistics update; cover at least the artifact mix, the documented branch/phase curves, the documented acceleration factors from agentic AI plus Spec-Kit/SDD support, and a direct comparison between experienced-developer effort, Thorsten-solo effort, and the visible AI-assisted delivery window.
- Keep each short CEFR-B2 explanation directly adjacent to its matching ASCII diagram group.
- When progression across an X-axis improves comprehension, add simple ASCII X/Y charts as a second visualization layer; keep them approximate, readable in plain Markdown, and explained in CEFR-B2 language.
- Keep the statistics section plain-text friendly for Braille displays, screen readers, and text browsers; diagrams and explanations must not rely on color or visual layout alone.
- When DocFX content, documentation navigation, or API presentation changes, validate representative `_site/` pages through a text-oriented review path, preferably with a local Playwright accessibility snapshot.
- Treat every successful `docfx` regeneration as requiring the matching text-oriented A11y smoke check in the same work item.
- Each update must capture branch or phase, observable work window, production/test/documentation line counts, main work packages, the conservative manual baseline of 80 manually created lines per workday across code, tests, and documentation, and the repo-specific Thorsten-Solo comparison baseline of 125 lines per workday for this Pascal-derived port.
- When effort is converted into months, use explicit assumptions such as 21.5 workdays per month and, if applicable, 30 vacation days per year through calendar year 2026 and 31 vacation days per year from calendar year 2027 onward under a TVoeD-style 5-day-week calendar.
- When reporting acceleration, compare both manual references against visible Git active days and label the result as a blended repository speedup rather than a stopwatch measurement.
- When hour values are shown, convert the day-based estimates with the TVoeD working-day baseline of `7.8 hours` (`7h 48m`) per day.

## Inclusion & Accessibility

- Programmierung #include<everyone> — Diese Lernbeispiele richten sich an Azubis (Fachinformatiker AE/SI) mit Deutsch und Englisch als Arbeitssprachen sowie an sehbehinderte Lernende, die mit Braille-Displays, Screen-Readern oder Textbrowsern arbeiten. Barrierefreiheit ist kein Nice-to-have, sondern Pflichtanforderung.
- Prefer semantic headings, lists, tables, and ASCII/text-first diagrams; do not encode essential meaning only through color, layout, or pointer-only affordances.
- Treat WCAG 2.2 conformance level AA as the concrete review baseline for generated HTML documentation, especially for page language, bypass blocks, keyboard focus visibility, non-text contrast, and readable landmark structure.
- If `docfx` output is regenerated, follow it with a text-oriented accessibility review, preferably with Playwright + `@axe-core/playwright` and a `lynx` cross-check.
- Recommended A11y toolchain for DocFX-based repos: Node 24 LTS, `npm`, Playwright, `@axe-core/playwright`, and `lynx`.
- Treat bilingual CEFR-B2 delivery and the documented A11Y proof path as formal completion criteria for learner-facing documentation and active requirement artifacts.

## Shared Parent Guidance

- Die gemeinsamen Dateien `/Users/thorstenhindermann/RiderProjects/AGENTS.md` und `/Users/thorstenhindermann/RiderProjects/GEMINI.md` speichern die repo-uebergreifenden Basisregeln.
- Diese Projekt-Datei ist die spezifischere Autoritaet fuer projektspezifische Build-Befehle, Workflows, Architektur und Features.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hindermath)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hindermath)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
