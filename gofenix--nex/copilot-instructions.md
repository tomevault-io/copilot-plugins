## nex

> Refer to `AGENTS.md` for the full set of principles.

# Cursor Rules for Nex

Refer to `AGENTS.md` for the full set of principles.

1. **Locality**: UI and logic belong in the same Page module.
2. **Routing**: File system based. No `router.ex`.
- **Navigation**: Use `hx-boost="true"` on the `<body>` tag for SPA-like speed.
- **CSRF Global**: Layout should include `{meta_tag()}` in `<head>` and `hx-headers={hx_headers()}` on `<body>` for HTMX requests.
3. **Actions**: Use `hx-post="/func_name"` to call module functions.
4. **API**: Use `Nex.json/2` in `src/api/`.

Always follow the Locality of Behavior (LoB) principle.

---
> Source: [gofenix/nex](https://github.com/gofenix/nex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
