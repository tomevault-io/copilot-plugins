## constants-conventions

> Store module-specific constants in colocated constants.js files


# Constants Conventions

- When a module accumulates non-trivial constants, move them into a colocated `constants.js` file instead of leaving them inline.
- Prefer importing constants from `./constants.js` or the nearest related `constants.js` for that module area.
- Keep behavior and helper functions in the main module; keep static config, IDs, labels, asset definitions, and similar constant data in `constants.js`.
- When adding new constants to an area that already has a related `constants.js`, extend that file instead of redefining the values locally.

---
> Source: [grebmann1/workbench](https://github.com/grebmann1/workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
