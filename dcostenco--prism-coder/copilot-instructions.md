## prism-coder

> See the Synalux engineering standards doc for the full protocol. This file inherits those standards.

# Engineering Standards

See the Synalux engineering standards doc for the full protocol. This file inherits those standards.
Additional Prism-specific rules:

1. **MCP Tool Testing:** When modifying tool handlers in src/tools/, run `npm test` before committing.
2. **Storage Backend Parity:** Changes to sqlite.ts must be mirrored in supabase.ts (and vice versa).
3. **Scheduler Safety:** backgroundScheduler.ts runs every 12h — never add blocking I/O to the sweep loop.

---
> Source: [dcostenco/prism-coder](https://github.com/dcostenco/prism-coder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
