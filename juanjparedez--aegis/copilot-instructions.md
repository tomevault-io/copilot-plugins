## aegis

> Read these files before making any changes:

Read these files before making any changes:

1. ESTADO.md (project root) — Current state, what's implemented, what's pending, how to run
2. docs/DEVELOPMENT.md — Development rules, conventions, technical decisions
3. docs/AEGIS-architecture.md — Full security architecture

Key rules:
- Ask before making design decisions
- Security tests BEFORE implementation
- Don't install unjustified dependencies
- One step at a time, each change must leave the project functional
- Run `npm test` (107 tests) before considering any change done
- NestJS requires explicit @Inject() — tsx doesn't emit decorator metadata
- Frontend is in web/ with its own package.json

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juanjparedez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
