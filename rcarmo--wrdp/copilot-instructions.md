## wrdp

> If `.agents.local.md` exists, read it before making changes. It contains machine- or workspace-specific instructions and is ignored by Git; never commit its contents.

# Repository Guidance for Agents

## Local Overrides

If `.agents.local.md` exists, read it before making changes. It contains machine- or workspace-specific instructions and is ignored by Git; never commit its contents.

## Engineering

- Prefer simple, local changes and apply YAGNI.
- Keep refactors mechanical before changing behavior.
- Preserve protocol behavior unless a task changes it explicitly.
- Keep RDP server, session, channel, desktop, service, security, and runtime ownership separate.
- Read relevant files before editing and validate with the Makefile.

## Documentation And Comments

- Treat documentation and comments as source code: review and rewrite them when moving code.
- Explain protocol constraints, ownership, invariants, negotiated capabilities, client quirks, and fallback reasons.
- Reference durable specifications or protocol names where known: MS-RDPBCGR, MS-RDPEGFX, MS-RDPECLIP, MS-RDPEA, Wayland, PipeWire, XDG Desktop Portal.
- Do not narrate obvious code, refactor history, private process, or repository preparation.
- Avoid generated prose: abstract value claims, generic capability brochures, repeated assurances, and headings such as “Overview” or “Conclusion”.
- Put consequential design choices in `docs/decisions/`.

## Rust

- Prefer concrete types until there are two real implementations or a useful test seam.
- Keep `unsafe` small, documented, and tied to a stated invariant.
- Use `Result` for recoverable errors; avoid `unwrap()` and `expect()` in production paths.
- Keep feature combinations used by the Makefile compiling.

## Commits

- Configure commits as `Rui Carmo <rui.carmo@gmail.com>`.
- Keep commits focused by architecture boundary.
- Run `git diff --check` before committing.
- Do not commit build output, logs, credentials, local paths, or local override files.

---
> Source: [rcarmo/wrdp](https://github.com/rcarmo/wrdp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
