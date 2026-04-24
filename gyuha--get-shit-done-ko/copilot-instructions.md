## get-shit-done-ko

> <!-- GSD:project-start source:PROJECT.md -->

<!-- GSD:project-start source:PROJECT.md -->
## Project

**get-shit-done-ko**

`get-shit-done-ko` is a Korean-localized fork of the upstream `get-shit-done` project, pinned to upstream `v1.28.0` as the initial source baseline. It should preserve the upstream repository structure and all command/file/path/identifier compatibility, while making Korean the default language for explanatory text, prompts, templates, comments, and other user-facing copy.

English documentation remains available, Simplified Chinese content is removed from this fork, and Korean becomes the primary reading path for day-to-day use.

**Core Value:** Korean-speaking users can use GSD end-to-end in Korean without breaking upstream command compatibility.

### Constraints

- **Upstream parity**: Start from upstream `v1.28.0` layout and preserve root-level structure — later maintenance depends on easy diffing against upstream.
- **Compatibility**: Commands, file paths, placeholders, and identifiers must remain unchanged — prompt routing and tooling expect exact tokens.
- **Language policy**: Korean is default, English remains available, Simplified Chinese is removed — this defines both content scope and navigation behavior.
- **Runtime safety**: Localization must not break `@` references, markdown links, snippets, CLI examples, or tests — translated prose cannot alter executable content.
- **Maintainability**: Translation choices should stay consistent across docs, prompts, templates, and comments — mixed terminology will confuse both users and downstream agents.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Baseline runtime
- **Node.js 20+** — upstream requires `>=20.0.0`; keep the same floor for install and test compatibility.
- **npm package distribution** — upstream ships as `get-shit-done-cc`; preserving packaging/layout avoids accidental runtime drift.
- **CommonJS Node scripts** — core tools and helpers live in `.cjs` files; avoid introducing a new module system during localization.
### Content surface
- **Markdown-first localization** — commands, workflows, templates, references, and docs are primarily `.md`; translation work should happen at source-file level rather than via runtime i18n wrappers.
- **Static asset parity** — keep images, hooks, scripts, and generated assets aligned with upstream unless language-specific copy requires a source edit.
- **Git-based upstream sync workflow** — treat upstream `v1.28.0` as the initial mirror baseline so future diffs remain understandable.
### Quality layer
- **Existing test suite** — upstream already includes Node-based tests; use them as regression protection after translation changes.
- **Reference integrity checks** — translation review must verify command tokens, `@` references, paths, placeholder syntax, and code snippets remain untouched.
## What Not To Change
- Do not rename command tokens such as `/gsd:*`, `$gsd-*`, flags, or file names.
- Do not add a runtime translation framework unless the source audit proves markdown-only localization is insufficient.
- Do not restructure directories while importing upstream; path stability is part of the product contract.
## Rationale
- The project is content-heavy rather than UI-heavy. Most behavior is encoded in text assets that downstream agents read directly.
- Markdown/source localization is lower risk than adding a second language-selection layer.
- Upstream parity makes future Korean maintenance and cherry-picking realistic.
## Confidence
- **Node/npm/CommonJS baseline** — High
- **Markdown-first localization approach** — High
- **Need for runtime i18n framework** — Low; current evidence suggests unnecessary complexity
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd:quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd:debug` for investigation and bug fixing
- `/gsd:execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd:profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gyuha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
