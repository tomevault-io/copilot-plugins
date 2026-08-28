## tagtoolbox

> Guidance for Claude Code when working in the **Tag Toolbox** repository.

# CLAUDE.md

Guidance for Claude Code when working in the **Tag Toolbox** repository.

## What this is

A standalone, MIT-licensed Unreal Engine plugin: a Gameplay Tag quality-of-life suite (designer-editable picker filters on Blueprint variables, project-wide tag colors, an enhanced dropdown picker with favorites/recents, a Tag Browser with a per-tag References pane, a no-load Tag Audit, collapsible container chip strips, and `#tag`-tinted graph comments). Two modules: `TagToolbox` (Runtime, settings only) and `TagToolboxEditor` (everything else). Nothing this plugin does runs in a cooked game.

Read `docs/architecture.md` before changing code — it carries the verified engine seams and the design rules; `docs/designer-guide.md` is the user-facing feature reference.

## Session start

Check `backlog/tasks/` for `In Progress` work first, then high-priority `To Do`. This repo uses the Backlog.md file convention (frontmatter + sectioned markdown); keep task status and notes current as work lands.

## Development workflow

This plugin has **no host project of its own**. Development happens submoduled into the LostRadiance repository at `Plugins/TagToolbox` (public origin: https://github.com/InfiniteGameWorksTeam/TagToolbox). When working from inside LostRadiance, that repo's CLAUDE.md governs builds (the shared UBT build broker — never invoke UBT directly), file-guard claims, and editor coordination; commits to this plugin are made in the submodule and pushed here.

- **Build (from the host)**: the host's broker with `-Target LostRadianceEditor -Engine 5.8`.
- **Headless tests**: `UnrealEditor-Cmd.exe <host.uproject> -ExecCmds="Automation RunTests TagToolbox; Quit" -TestExit="Automation Test Queue Empty" -nullrhi -unattended -nop4 -NoSplash -ReportExportPath=<dir>` → parse `index.json`; require queue completion and zero failures. Don't pin a fixed test count.
- **Live verification seams**: console commands `TagToolbox.OpenTagBrowser` and `TagToolbox.OpenTagAudit`; features are verifiable end-to-end with a throwaway probe Blueprint (create tag/container variables via Python's `BlueprintEditorLibrary` + `get_struct_type`/`get_array_type`, delete after).

## Non-negotiable code rules

The full list with rationale is in `docs/architecture.md` §Design rules. The ones that have already drawn blood:

1. Never name a local/parameter `Tag` inside an `SWidget` subclass (C4458 vs `SWidget::Tag` under warnings-as-errors).
2. Rebuilds triggered from a widget's own click handler defer one tick (`RegisterActiveTimer`) — never `ClearChildren` the widget whose handler is on the stack.
3. Menu-hosted pickers are selection-only and RMB-guarded (`STagToolboxMenuHostedPickerGuard`) — a context menu pushed from an auto-dismissing menu host fatal-asserts.
4. Resolve `Categories` filters via `UGameplayTagsManager::GetCategoriesMetaFromPropertyHandle`, never bare `GetMetaData`.
5. Property commits mirror the engine widgets' exact transaction + write shapes.
6. Config: `TArray` of entries (never tag-keyed `TMap`), no CDO-constructor tag seeding, no manual `GConfig` flushes on interactive paths.
7. Registration of the editor-wide type customizations stays one-tick-past-PostEngineInit, with the Paper2DPlus yield probe for the single-tag pill.
8. Test helpers carry file-unique prefixes (unity builds), and any test that opens asset editors must early-return under `!FApp::CanEverRender()`.

## Versioning

Semver via `TagToolbox.uplugin` `VersionName` + `Version` and `CHANGELOG.md`, moved together in one commit. Currently pre-1.0 (`0.1.0` unreleased): breaking changes are acceptable but must be changelogged. From 1.0 on: removed/renamed public symbols, settings fields, or designer-visible behavior contracts force a MAJOR.

## Cross-version work

v1 is UE 5.8-only. The gate table for 5.0–5.7 lives in `docs/architecture.md` and backlog TASK-2 — engine-version guards must be verified against real installed engine source, not assumed.

---
> Source: [InfiniteGameWorksTeam/TagToolbox](https://github.com/InfiniteGameWorksTeam/TagToolbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
