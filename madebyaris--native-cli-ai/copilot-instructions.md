## project-docs

> Rules for writing and maintaining project documentation.


# Documentation Standards

## Structure

- `docs/prd.md` is the source of truth for product scope, personas, and MVP definition.
- `docs/tech-stack.md` records every dependency choice with rationale and rejected alternatives.
- `docs/architecture.md` defines crate boundaries, data flows, IPC protocol, and security model.
- The root `README.md` is a quick-start guide. It links to the docs folder for details.

## Writing Style

- Use plain, direct language. Avoid marketing copy.
- Use tables for comparisons and structured data.
- Use mermaid diagrams for architecture, data flows, and state machines.
- Keep each document under 500 lines. Split into separate files if it grows beyond that.

## Consistency

- When a dependency is added or removed, update `docs/tech-stack.md` in the same commit.
- When a crate boundary changes, update `docs/architecture.md` in the same commit.
- When MVP scope changes, update `docs/prd.md` in the same commit.
- Cursor rules in `.cursor/rules/` must stay in sync with architectural decisions in the docs.

## Versioning

- Docs are version-controlled alongside code.
- `.nca/config.local.toml` and `.nca/sessions/` are gitignored. Never document them as committed files.
- `.ncarc` (project instructions) is version-controlled and documented.

---
> Source: [madebyaris/native-cli-ai](https://github.com/madebyaris/native-cli-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
