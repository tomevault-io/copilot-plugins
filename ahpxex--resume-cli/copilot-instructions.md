## resume-cli

> This project is a Bun-based CLI for generating resumes from structured input data.

# AGENTS.md

## Project Overview

This project is a Bun-based CLI for generating resumes from structured input data.

The product direction is:

- Built-in style templates ship with the CLI.
- A resume is composed from reusable sections such as `education`, `experience`, `projects`, `skills`, and similar structured blocks.
- The CLI should help users generate polished resumes without forcing them to hand-author layout code.

## Stack Decisions

- Runtime: `bun`
- Language: `TypeScript`
- Module system: ESM
- CLI parser and command framework: `cac`

Prefer Bun-native workflows unless there is a clear blocker.

- Use `bun install`, `bun run`, `bun test`, and `bun build`.
- Avoid `npm`, `pnpm`, or `yarn` unless there is a concrete compatibility reason.
- Keep dependencies small and focused. Do not add large CLI frameworks now that `cac` is the chosen command layer.

## Product Principles

- Keep the user-facing CLI simple, predictable, and scriptable.
- Prefer explicit flags and file-based input over hidden behavior.
- Separate resume data from presentation.
- Separate template concerns from core resume schema concerns.
- Preserve deterministic output for the same input and template.

This should feel like a reliable content-generation tool, not an interactive wizard-first app.

## Domain Model Expectations

Treat resume data as a canonical internal document model, even if we support multiple input formats later.

At minimum, design with these concepts in mind:

- Resume metadata: name, title, contact details, links, summary
- Core sections: `education`, `experience`, `projects`, `skills`
- Optional sections: certifications, awards, publications, volunteering, custom sections
- Templates: built-in presentation/layout variants that render the same normalized resume data differently

Important rule:

- Templates should not invent their own data schema.
- Input data should be normalized into one internal model before rendering.
- Adding a new section should not require rewriting the whole rendering pipeline.

## Architecture Guidelines

Keep the codebase split by responsibility.

- `src/cli/*`: command registration, flags, help text, and thin command handlers
- `src/domain/*`: canonical resume types and transformation logic
- `src/schema/*`: input validation and parsing
- `src/templates/*`: built-in template definitions and template-specific rendering
- `src/render/*`: output generation helpers shared across templates
- `src/io/*`: file loading, writing, and path handling

Guidelines:

- Command handlers should stay thin and call reusable domain/rendering functions.
- Validation and normalization should happen before rendering.
- Template files should focus on presentation, not business rules.
- Avoid coupling one template to another.
- Prefer pure functions for transforms and rendering where practical.

## CLI UX Guidelines

Every command should have strong defaults, clear help text, and useful errors.

- Provide `--help` output that includes concrete examples.
- Error messages should explain what failed and what the user can do next.
- Prefer stable, descriptive option names over short clever aliases.
- Keep commands non-interactive by default so they work in scripts and CI.
- If interactive features are added later, they should be optional and layered on top.

When designing commands, favor a flow like:

- load input
- validate and normalize
- choose template
- render output
- write file or print result

## Template Guidelines

Built-in templates are a core feature, so keep them easy to extend.

- Each template should have a stable identifier and human-friendly label.
- Templates should share the same normalized input model.
- Shared formatting helpers should live outside individual template files.
- Avoid hardcoding section-specific logic in multiple places.
- Template output should be consistent and testable.

If we support multiple output targets later, keep template selection separate from output format concerns.

## Quality Bar

Use `bun test` for automated tests.

Prioritize tests for:

- input validation
- normalization
- section ordering and rendering behavior
- template output for representative resume fixtures
- regression coverage for edge cases like missing optional fields

When behavior is highly presentational, snapshot-style tests are acceptable if they stay readable and intentional.

## Documentation Expectations

Keep the docs close to the product.

- Update `README.md` when commands, flags, or input formats change.
- Add runnable examples when introducing a new command or template.
- Document assumptions in code when a decision is not obvious from the implementation.

## Near-Term Bias

For the next phase of the project, optimize for:

- one clear canonical resume schema
- a small but solid command surface
- a clean template abstraction
- easy addition of new built-in templates

Avoid premature plugin systems, remote template registries, or overly dynamic configuration until the core CLI experience is stable.

---
> Source: [ahpxex/resume-cli](https://github.com/ahpxex/resume-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
