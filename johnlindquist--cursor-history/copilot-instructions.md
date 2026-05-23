## cli-execution

> To run the Cursor History (chi) CLI or any of its commands, **ALWAYS USE** the provided scripts in [package.json](mdc:package.json) via `pnpm`:

# CLI Execution Rule

To run the Cursor History (chi) CLI or any of its commands, **ALWAYS USE** the provided scripts in [package.json](mdc:package.json) via `pnpm`:

- Use `pnpm default` to run the default command
- Use `pnpm extract` to extract all conversations
- Use `pnpm search` to search conversations
- Use any other scripts defined in [package.json](mdc:package.json)

**Do NOT run** `node ./bin/run.js` directly. This can cause native module version mismatches with sqlite/better-sqlite3 and break the CLI.

> Example:
> ```sh
> pnpm default
> pnpm extract
> pnpm search
> ```

**Reference:** [package.json](mdc:package.json) for all available scripts.

---
> Source: [johnlindquist/cursor-history](https://github.com/johnlindquist/cursor-history) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
