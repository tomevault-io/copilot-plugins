## backend

> How servers in this repo are installed and run


# Backend

- Use `pnpm`. Do not use `npm` or `yarn` for these servers.
- Each server has its own `package.json`.
- Phase 1 is vertical only: 1 instance, 1 port, cluster workers inside that process.
- Run one server at a time. Start it, test it, stop it, then start the next. Do not run the four servers together.
- Do not kill a process yourself. Ask the user, or let the user run the bench.
- Horizontal scaling, extra machines, and nginx are out of phase 1.

---
> Source: [xDAnkit/system-design-journey](https://github.com/xDAnkit/system-design-journey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
