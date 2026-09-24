## droppykit

> Tmdb is a Droplet for Droppy, built with DroppyKit. The brief, the build loop and the rules live in AGENTS.md at the package root.


Read `AGENTS.md` at the package root before changing anything in this package. It carries the loop (`droppykit build`, `droppykit validate`, `droppykit run -- --shots ./shots --report ./shots/report.json`), the rules a droplet must follow, and the on-disk paths of the DroppyKit guides and sources under `/Users/jasneet/droppykit`.

Never build the bundle with a bare `swift build`; only `droppykit build` produces one Droppy can load. Keep `droplet.json` `surfaces` and the droplet's protocol conformances in agreement. After every visual change render the shots and look at them: the DroppyKit MCP server in `.cursor/mcp.json` returns them inline through `droppykit_shots`, and `droppykit_install` puts the bundle into Droppy Playground and reports whether it loaded.

---
> Source: [ShubhamPP04/cinema-droplet](https://github.com/ShubhamPP04/cinema-droplet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
