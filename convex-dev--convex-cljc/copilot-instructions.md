## convex-cljc

> Code throughout the whole repository follows the following conventions.

# Conventions

Code throughout the whole repository follows the following conventions.


---


## Directory structure

- Each module has a directory in [./module](../module) with `./src` and documentation
- `./src` is divided
    - First by purpose
        - `./main`
        - `./test`
        - etc
    - Then by language
        - `./clj`
        - `./cvx`
        - etc


---


## Naming

- Namespace aliases are as close to their actual symbol as possible
- Namespaces from this repository starts always with `convex.`
    - First segment is aliased as `$.`
    - Rest of the symbol is kept intact
    - E.g. `convex.cvm` -> `$.cvm`
- Pluralize with `+` instead of `s`
    - E.g. `items` -> `item+`

---
> Source: [Convex-Dev/convex.cljc](https://github.com/Convex-Dev/convex.cljc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
