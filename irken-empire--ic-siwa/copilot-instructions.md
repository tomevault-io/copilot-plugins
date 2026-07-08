## ic-siwa

> All AI agent instructions, technology stack guidance, and project conventions are documented in a single source of truth:

# Gemini Code Instructions

All AI agent instructions, technology stack guidance, and project conventions are documented in a single source of truth:

**→ See [`AGENTS.md`](./AGENTS.md)**

This file contains:

- Quick start commands and helper scripts
- Project overview and repository structure
- AI agent documentation references
- Technology stack (Rust, Astro, TypeScript, DaisyUI)
- Development workflow and devenv setup
- Code standards and commit conventions
- Agent-specific instructions

**Quick Start:**

```bash
ic-siwa build         # Build all canisters and libraries
ic-siwa deploy        # Deploy to local dfx replica
ic-siwa test          # Run tests
ic-siwa help          # Show all available commands
```

For devenv shell access in AI sandboxes:

```bash
SECRETSPEC_PROVIDER=env devenv shell --quiet -- <command>
```

See `AGENTS.md` for complete details.

---
> Source: [irken-empire/ic-siwa](https://github.com/irken-empire/ic-siwa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
