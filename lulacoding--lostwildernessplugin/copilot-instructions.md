## lostwildernessplugin

> Core project conventions and structure for Lost Wilderness


# Lost Wilderness – Project Conventions

## Docs as source of truth
- Design and implementation status live in `Docs/` and `Docs/roadmap/implementation-status.md`.
- For large or cross-cutting work, read `Docs/architecture/v2-architecture.md` when touching architecture.

## After new features
- Log changes in `CHANGELOG.md` and, when relevant, `Docs/roadmap/implementation-status.md`.

## Package and modules
- **Current plugin:** `com.lula0802.LWEvents` under `Plugin/` (common, survival, amplified, lobby, portals, etc.).
- **V2 work:** lives under `PluginV2/` and the planned `com.lostwilderness.rpgcore` package.
- Do not mix V2 patterns into the legacy `Plugin/` tree unless explicitly refactoring.

## Stack
- Java 21, Paper API 1.21.x, Gradle.
- Multi-module layout: `Plugin/common`, `Plugin/servers/survival`, `Plugin/servers/amplified`, `Plugin/servers/lobby`, etc.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lulacoding) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
