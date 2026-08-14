## adobe-plugin-skills

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

An AI agent skill (following the [agentskills.io](https://agentskills.io) specification) that provides SDK knowledge for building Adobe Photoshop UXP plugins and Lightroom Classic plugins. There is no build system, no source code, and no tests — the deliverable is the markdown files themselves.

## Repository Structure

- **`SKILL.md`** — Skill entry point with YAML frontmatter (500 lines — keep at or under the agentskills.io 500-line recommendation). The "90% file" that loads on every activation. Contains capability matrix, critical pitfalls, quick-reference code snippets (incl. file I/O + persistence), Info.lua essentials, data binding patterns, and common mistakes table. The develop-parameter/LrDevelopController catalogs and the full Info.lua table live ONLY in `lrc-sdk.md` — don't re-duplicate them here.
- **`ps-uxp-sdk.md`** — Deep Photoshop UXP reference (~1390 lines), organized by use-case. Covers Document/Layer DOM model, batchPlay, Imaging API + ImageBlob, File I/O & storage (pickers, tokens, secureStorage), XMP, Spectrum components, CSS support, Text/Color APIs, executeAsModal, `.psjs` scripts, photoshop.constants, UDT workflow, path creation, hybrid C++ bridge, and 37 known issues.
- **`lrc-sdk.md`** — Deep Lightroom Classic SDK reference (~1220 lines), organized by use-case. Covers plugin architecture, export/publish pipeline (incl. programmatic export via LrExportSession and the full publish callback catalog), memory leak prevention, LrCatalog method inventory, LrPhoto APIs, plugin develop presets, LrView/LrDialogs, data binding, 95 LrDevelopController functions, and 45+ module quick reference.
- **`README.md`** — Installation instructions and project overview for GitHub.

## SDK Versions Covered

- UXP v9.2.0, Manifest v5
- Photoshop API changelog 27.4 (verified still current as of July 2026 — the app is at 27.8 but Adobe has documented no plugin-API changes since 27.4; the 27.8 Beta began a UXP plugin-rendering replatform, announced 2026-05-12, with no documented API changes)
- Lightroom Classic SDK v15.3 (latest published; the LrC 15.4 app, June 2026, shipped without a new SDK)

## Reference Snapshots (docs/, gitignored)

- `docs/references/ps-uxp-sdk/` — local clone of [AdobeDocs/uxp-photoshop](https://github.com/AdobeDocs/uxp-photoshop). Update with `git pull`. Since the May 2026 EDS replatform, paths are kebab-case (`src/pages/ps-reference/...`). This repo also embeds the UXP-level API docs under `src/pages/uxp-api/`.
- `docs/references/lrc-sdk/` — the LrC SDK distribution (LuaDoc HTML in `API Reference/`, Guide PDF in `Manual/`). New SDKs are published on the [Adobe Developer Console](https://developer.adobe.com/console) (requires login) — check there when a new LrC major/MAX release ships.
- `docs/references/uxp/` — clone of [AdobeDocs/uxp](https://github.com/AdobeDocs/uxp); dormant since April 2025 (UXP docs moved into the per-app repos). Historical reference only.

## Editing Guidelines

- Content was compiled from Adobe's official docs ([AdobeDocs/uxp-photoshop](https://github.com/AdobeDocs/uxp-photoshop), [AdobeDocs/uxp](https://github.com/AdobeDocs/uxp)) and the LrC 15.3 SDK, supplemented with lessons from building [Chromascope](https://github.com/kevinkiklee/chromascope).
- When updating API references, verify against Adobe's current documentation — APIs change between SDK releases.
- **Currency signal:** the file to watch is `docs/references/ps-uxp-sdk/src/pages/ps-reference/changelog/index.md` (the PS API changelog). Photoshop app releases outpace the plugin API — don't bump the skill's baseline for app releases that add no changelog entry. For LrC, watch for SDK announcements on community.adobe.com and the Developer Console.
- `SKILL.md` is the agent-facing "90% file" — it should cover most common use cases without needing the deep references. Deep details and complete API tables belong in `ps-uxp-sdk.md` or `lrc-sdk.md`, which are organized by use-case.
- The YAML frontmatter in `SKILL.md` (`name`, `description`) controls how agent frameworks discover and trigger this skill. Changes there affect activation behavior. Per the agentskills.io spec, only `name`/`description`/`license`/`compatibility`/`metadata`/`allowed-tools` are valid root fields — the release version lives at `metadata.version`.

## Installation (for reference)

Installed as a Claude Code skill by cloning/symlinking into `~/.claude/skills/adobe-plugin-development`. No dependencies to install.

---
> Source: [kevinkiklee/adobe-plugin-skills](https://github.com/kevinkiklee/adobe-plugin-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
