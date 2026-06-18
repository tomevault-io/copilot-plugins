## avalonia-codex-plugin

> Umbrella skill for building, reviewing, designing, porting, and migrating Avalonia 12 applications with modern XAML/C# patterns. Use when the request is broad Avalonia 12 work or when another specialist skill has not yet been selected; route quickly to focused skills for startup, bindings, styling, controls, layout, rendering, testing, design systems, or HTML/WinForms/WPF/WinUI/legacy Avalonia migration work.


# Avalonia Codex Plugin

Use this as the canonical umbrella workflow source for broad Avalonia work in this repository.

Discovery entrypoints:

- repo-local skill: `.agents/skills/avalonia-codex-plugin/SKILL.md`
- plugin discovery: focused skills under `skills/` via `.codex-plugin/plugin.json`

Do not treat this root file as the repo-local discovery entrypoint. Keep it as the canonical routing workflow that the repo-local wrapper can load without duplicating the full guidance.

Resolve the task category, load the smallest specialist skill that fits, and keep the shared references as the single source of truth.

Primary shared indexes:

- `references/compendium.md`
- `references/70-avalonia-12-source-and-reference-baseline.md`
- `references/71-skill-routing-and-evaluation.md`
- `references/73-avalonia-12-ai-desktop-product-patterns.md` when the task involves AI desktop product patterns, workbenches, assistants, trays, overlays, settings centers, plugins, or MCP surfaces.
- `references/74-avalonia-12-ai-desktop-recipes-and-checklists.md` when the task needs AI desktop recipes, acceptance checklists, eval recording, or product-pattern QA.
- `references/00-api-map.md`
- `references/api-index-generated.md` as broad signature lookup regenerated from the local Avalonia 12 source tree.

## Default Working Rules

- Keep default implementation guidance pinned to the local Avalonia 12 source baseline at `/Volumes/程序开发/Du-Framework/Du.Ingest/frameworks/Avalonia`.
- Treat `references/68-avalonia-12-migration-guide.md` as a legacy-app migration aid, not as the default Avalonia 12 guidance source.
- Use only the Avalonia 12 projects listed in `references/70-avalonia-12-source-and-reference-baseline.md` as default product references.
- Prefer XAML-first examples unless the user explicitly asks for code-only UI construction.
- Prefer compiled bindings with `x:DataType`.
- Keep UI-thread work explicit and keep AOT/trimming tradeoffs visible.
- Keep evidence classes separate: Avalonia 12 source facts, Avalonia 12 project patterns, and Avalonia 11.x migration contrast.

## Capability Lanes

Use these capability lanes to keep routing testable:

- `umbrella`: classify intent and route to the narrowest specialist skill.
- `app-building`: create or modify Avalonia 12 apps using source-backed APIs and current `net10.0` assumptions.
- `migration`: port legacy Avalonia, WPF, WinForms, WinUI, or HTML/CSS inputs into Avalonia 12.
- `debugging`: diagnose build, XAML, binding, styling, rendering, platform, or performance failures.
- `source-reference`: verify uncertain APIs and behavior against `frameworks/Avalonia`.
- `ui-patterns`: extract architecture and UI patterns only from allowed Avalonia 12 reference projects; load `references/73-avalonia-12-ai-desktop-product-patterns.md` and `references/74-avalonia-12-ai-desktop-recipes-and-checklists.md` for AI desktop product-mode work.

## Routing Rules

Route to the first specialist skill that matches the request and do not keep broad orchestration in scope longer than needed.

- Startup, `AppBuilder`, platform entrypoints, lifetimes, build configuration:
  `skills/avalonia-bootstrap-and-lifetime/SKILL.md`
- Compiled bindings, runtime XAML, converters, dynamic resources, AOT-safe markup:
  `skills/avalonia-bindings-and-xaml/SKILL.md`
- Reactive flows, dispatcher usage, timers, UI-thread correctness:
  `skills/avalonia-threading-and-dispatcher/SKILL.md`
- Styles, themes, resources, property system, asset packaging:
  `skills/avalonia-styling-and-resources/SKILL.md`
- View location, templates, templated parents, tree traversal:
  `skills/avalonia-views-and-templating/SKILL.md`
- Input, commands, focus, gestures, drag/drop, text editing:
  `skills/avalonia-input-and-commands/SKILL.md`
- Controls, popups, menus, windows, tray, notifications:
  `skills/avalonia-controls-and-windowing/SKILL.md`
- Layout, panels, measure/arrange, virtualization, large item surfaces:
  `skills/avalonia-layout-and-virtualization/SKILL.md`
- Animation, compositor, drawing, Skia, rendering interop:
  `skills/avalonia-rendering-and-graphics/SKILL.md`
- File pickers, clipboard, launcher, screens, platform integration:
  `skills/avalonia-platform-services/SKILL.md`
- Validation, accessibility, automation semantics:
  `skills/avalonia-accessibility-and-validation/SKILL.md`
- Tests, diagnostics, profiling, troubleshooting, performance hardening:
  `skills/avalonia-testing-diagnostics-and-performance/SKILL.md`
- Professional design systems, tokens, motion, dense workflow UX:
  `skills/avalonia-design-systems/SKILL.md`
- Microsoft Fluent design, `FluentTheme`, palette and shell guidance:
  `skills/avalonia-fluent-design/SKILL.md`
- HTML/CSS to Avalonia migration:
  `skills/html-css-to-avalonia/SKILL.md`
- WinForms to Avalonia migration:
  `skills/winforms-to-avalonia/SKILL.md`
- WPF to Avalonia migration:
  `skills/wpf-to-avalonia/SKILL.md`
- WinUI to Avalonia migration:
  `skills/winui-to-avalonia/SKILL.md`
- Legacy Avalonia 11 to Avalonia 12 migration planning and execution:
  `skills/avalonia-12-migration/SKILL.md`

## First Pass Workflow

1. Identify whether the user is building new UI, fixing an existing app, or porting from another stack.
2. Pick the narrowest specialist skill that fits the task.
3. Load only the reference documents that specialist skill points to.
4. For any broad or design-heavy task, load `references/70-avalonia-12-source-and-reference-baseline.md` before using third-party product references; for AI desktop workbench, assistant, overlay, tray, settings, plugin, MCP, notification, or clipboard utility tasks, also load `references/73-avalonia-12-ai-desktop-product-patterns.md` and `references/74-avalonia-12-ai-desktop-recipes-and-checklists.md`.
5. Use generated API indexes only when signature-level lookup is required; verify important signatures against `/Volumes/程序开发/Du-Framework/Du.Ingest/frameworks/Avalonia`.
6. Escalate to the legacy migration lane only when the request explicitly involves upgrading older Avalonia code.
7. When validating the plugin itself, run `python3 scripts/validate_plugin_quality.py`, `python3 scripts/run_ai_desktop_eval_checklist.py --check`, and inspect `evals/avalonia-12-plugin-prompts.md`.

## Output Expectations

- For broad requests, state which specialist path you are taking and why.
- For implementation work, keep architecture, view/viewmodel, and styling concerns separated.
- For migration work, anchor comparisons in the source framework's idioms and call out Avalonia equivalents explicitly.

---
> Source: [dusdong/avalonia-codex-plugin](https://github.com/dusdong/avalonia-codex-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
