## byo-ai-grammar

> This file gives coding agents and automation tools project-specific guidance for `byo_ai_grammar`.

# AGENTS.md

This file gives coding agents and automation tools project-specific guidance for `byo_ai_grammar`.

## Project Summary

`byo_ai_grammar` is a Thunderbird Manifest V3 add-on that adds inline grammar suggestions while composing email.

Important product boundaries:

- Grammar only, not spelling
- Thunderbird native spelling remains the source of truth for spelling and personal dictionary behavior
- OpenAI-compatible backend, with Together.ai as a common target
- User provides their own endpoint, model, and key

## Core Product Decisions

Preserve these unless there is an explicit product decision to change them:

- Do not reintroduce LLM-based spelling checks as a first-class feature.
- Keep the add-on grammar-focused and conservative.
- Treat Thunderbird native spellcheck as complementary, not something to replace.
- Keep inline UX lightweight and robust rather than overly clever.
- Prefer bounded paragraph-level or nearby-paragraph context instead of whole-message live checking.

## Repository Layout

- `public/` - static extension assets such as `manifest.json`, `background.html`, options UI HTML/CSS, icons
- `src/background/` - background page logic, settings, menus, compose-script registration, LLM client
- `src/compose/` - compose-window DOM logic, block extraction, highlights, popup UI, range mapping
- `src/options/` - options page TypeScript
- `src/shared/` - shared types, prompt logic, validation, and shared constants
- `scripts/` - local tooling such as `.xpi` packaging
- `.github/workflows/` - GitHub Actions workflows

## Local Commands

- `npm install` - install dependencies
- `npm run build` - build extension into `dist/`
- `npm run test` - run focused unit tests
- `npm run watch` - rebuild on change
- `npm run typecheck` - run TypeScript checks
- `npm run package` - build and create an installable `.xpi`

## Thunderbird Install Paths

Use one of these during development:

- Temporary dev load: select `dist/manifest.json` in Thunderbird Debug Add-ons
- Local installable package: run `npm run package` and install the generated `.xpi`

Do not assume Thunderbird can install directly from source files outside `dist/`.

## Architecture Notes

### Background Page

The background layer is responsible for:

- settings storage
- compose-script registration
- OpenAI-compatible network requests
- response validation and filtering
- request cancellation and stale-response protection
- per-tab pause state
- menu actions

Background code must tolerate restart/reload behavior and should keep setup idempotent.

### Compose Script

The compose script is responsible for:

- identifying the active block and nearby context
- sending bounded check requests to the background page
- preserving block snapshots long enough to reject stale responses
- rendering inline highlight overlays
- showing the suggestion popup
- applying accepted replacements

Be careful with DOM changes. Do not let extension UI leak into outgoing message content.

### Prompt And Validation

Prompting lives in `src/shared/prompt.ts`.

Local diffing lives in `src/shared/diff.ts`.

Guardrails:

- keep prompts short and simple enough for smaller models
- target contemporary standard English with light formality
- preserve names, product names, quoted text, and meaning
- require strict JSON output
- prefer `corrected_text` plus local diffing instead of model-provided offsets
- validate all model output before use

## API Key Handling

One supported mode exists:

- saved key in Thunderbird `browser.storage.local`

Important:

- local storage should not be described as high-security secret storage

Do not remove or soften the security warnings in the UI or docs without good reason.

## Coding Standards

### TypeScript

- Keep TypeScript `strict`-friendly
- Prefer explicit shared types for messages and settings
- Avoid `any` unless there is a clear Thunderbird API typing gap
- Prefer defensive parsing around network responses and editor state
- Prefer snapshot-based reconciliation and last-write-wins request handling for compose checks
- Document code for an intermediate TypeScript developer who has only basic familiarity with Thunderbird extensions
- Follow normal TypeScript community practice by using concise TSDoc-style doc comments for exported functions, types, and non-obvious helpers where intent, constraints, or Thunderbird-specific behavior may not be clear from the code alone
- Favor comments that explain why, assumptions, lifecycle constraints, and Thunderbird quirks rather than restating obvious code flow
- Add or update inline documentation when changing non-obvious logic, especially around compose DOM behavior, background lifecycle, prompt and validation rules, and request reconciliation

### UI

- Keep copy plain, direct, and beginner-friendly
- Use the established naming consistently: `BYO AI Grammar`, `grammar suggestions`, `allowlist`
- Do not introduce a frontend framework for small UI changes
- Keep options and popup UI usable on narrow windows
- Keep verbose troubleshooting logs behind the debug mode setting

### Packaging And Release

- Keep `npm run package` working on Linux, macOS, and Windows
- Keep `npm run test` fast and focused on pure logic
- Keep `manifest.json` at the root of the generated `.xpi`
- Keep `package.json` and `public/manifest.json` versions aligned
- If release packaging changes, update both `README.md` and the GitHub Action workflow

## Documentation Requirements

When behavior changes, update the relevant docs:

- `README.md` for setup, installation, configuration, and release workflow changes
- `CONTRIBUTING.md` for workflow or coding-standard changes
- `CHANGES.md` for release-facing user-visible changes

Code documentation expectations:

- Keep code comments and doc comments aligned with the current implementation when behavior changes
- Prefer TSDoc-style comments on exported APIs and shared utilities that are likely to be read by contributors first
- Document Thunderbird-specific assumptions, browser API gaps, and DOM invariants close to the code that depends on them

Documentation patterns to follow:

- Module-level example:

```ts
/**
 * Compose-side helpers for mapping Thunderbird editor text offsets back to DOM ranges.
 *
 * Thunderbird compose markup is not stable across plain-text and HTML modes, so callers
 * should treat these helpers as best-effort and always validate returned ranges.
 */
```

- Function-level example:

```ts
/**
 * Returns the visible text offset for the current selection start within the compose body.
 *
 * Thunderbird may place the caret inside wrapper elements that do not correspond to the
 * outgoing message HTML, so this walks visible text rather than trusting DOM depth alone.
 */
export function getSelectionVisibleOffset(): number | null {
  // ...
}
```

- Inline comment example:

```ts
// Ignore extension-owned overlay nodes here so suggestion UI never leaks into
// the text snapshot used for prompts or outgoing-message DOM reconciliation.
```

## Things To Avoid

- Re-adding spelling corrections as a normal suggestion path
- Sending whole-message draft content by default while typing
- Expanding prompt size carelessly
- Introducing telemetry or analytics
- Claiming secure secret storage where none exists
- Breaking cross-platform packaging for `.xpi` generation

## Good Next-Step Areas

Useful future work includes:

- improving plain-text compose fallback behavior
- improving overlay stability during scroll or layout changes
- refining grammar allowlist behavior
- improving release automation while keeping local packaging simple

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/timrichardson)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/timrichardson)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
