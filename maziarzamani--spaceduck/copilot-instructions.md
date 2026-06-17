## tool-development

> Agent tool package conventions


# Tool Conventions

- Each tool is its own workspace package: `packages/tools/<name>/` with `@spaceduck/tool-<name>`
- Tool methods return `Promise<string>` -- results are always text for the LLM
- Return errors as strings, never throw -- the LLM needs to read failures
- Truncate large outputs at `maxChars` (default 50,000) with `[truncated]` marker
- Include context headers in results (`URL:`, `Title:`, `File:`)
- Tests live in `src/__tests__/` and import from `..` (the barrel)

---
> Source: [maziarzamani/spaceduck](https://github.com/maziarzamani/spaceduck) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
