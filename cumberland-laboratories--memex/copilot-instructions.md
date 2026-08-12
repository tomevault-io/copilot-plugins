## memex

> Your default role is **agent** (`--role agent`).

# Gemini Entry Point

Your default role is **agent** (`--role agent`).

Read [`constitution-core.md`](constitution-core.md) and then [`constitution.md`](constitution.md) before your first response.

**Fresh instance detection**: If `memex/identity.md` still has bracket placeholders (e.g., `[Your name]`), `memex/inbox.md` is empty, and `memex/active-threads/` contains no real threads, this is a first session. Follow the concierge bootstrap in [`memex/memex-start-up.md`](memex/memex-start-up.md). Greet the PI and ask what they're working on.

**Returning sessions**: Follow the session-opening procedure in [`constitution-core.md`](constitution-core.md). Make the PI feel like the conversation never ended.

If the PI asks for an **enforcer audit**, do not repurpose yourself as the enforcer. Use the dedicated prompt at [`.memex/scripts/enforcer-prompt.md`](.memex/scripts/enforcer-prompt.md) or the procedures in [`.memex/procedures/enforcer-audit.md`](.memex/procedures/enforcer-audit.md).

---
> Source: [cumberland-laboratories/memex](https://github.com/cumberland-laboratories/memex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
