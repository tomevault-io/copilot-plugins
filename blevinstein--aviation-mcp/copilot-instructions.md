## project-rules

> NEVER create a separate jest.config file, always put these in package.json.

NEVER create a separate jest.config file, always put these in package.json.
NEVER `cd` to another directory, always stay in the project root.
NEVER use CommonJS, use ES6 JS for everything as much as possible.
NEVER use npm, only yarn.
NEVER swallow errors, or use a "fallback" codepath in case of errors, or do anything else that might fail silently.
ALWAYS fail quickly safely and visibly.

---
> Source: [blevinstein/aviation-mcp](https://github.com/blevinstein/aviation-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
