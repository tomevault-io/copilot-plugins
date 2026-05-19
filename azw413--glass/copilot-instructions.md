## glass

> Any Rust module over **1500 lines** should be refactored and split into

# Glass — project notes

## Module size

Any Rust module over **1500 lines** should be refactored and split into
smaller modules. `glass-ui/src/lib.rs` has historically violated this
and is being progressively modularised; new code should not regrow it.

## Skill catalog parity

`glass-api/src/skills.rs` is the single source of truth for the
automation API surface that drives both the CLI subcommands and the
MCP server (`glass-mcp`). When you add, rename, or remove a verb,
update **all four** in the same change:

1. The `glass_api` function / method that does the work.
2. The CLI subcommand wiring in `glass-cli/src/main.rs` +
   `verbs.rs` (clap derive + dispatch arm + text renderer).
3. The `Skill { … }` entry in `glass-api/src/skills.rs` (name must
   match the CLI subcommand name kebab-cased).
4. The dispatch arm in `glass-mcp/src/dispatch.rs` (matches on the
   same kebab-case name; extracts args from the JSON object).

Also update `docs/cli-api.md` to keep the user-facing reference in
sync. Skipping the catalog or the MCP dispatcher is the failure
mode that bites here — the CLI keeps working while LLM-driven
tooling silently loses the verb.

---
> Source: [azw413/Glass](https://github.com/azw413/Glass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
