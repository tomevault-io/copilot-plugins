## granite-cli

> - Avoid adding new dependencies if an existing dependency or standard library could serve the same function

## Project Practices

- Avoid adding new dependencies if an existing dependency or standard library could serve the same function
- Use the minimal visibility scoping for every function (only `pub` if necessary, crate-pub unless full pub necessary)
- When laying out a source file, organize it as follows:
    1. `use` directives
    2. Public functions/structs/enums/etc (mark with `/*-- public --*/` to begin section)
    3. Private functions/structs/enums/etc (mark with `/*-- private --*/` to begin seciton)
    4. Tests (mark with `/*-- tests --*/` to begin seciton)
- When formatting code, use `./scripts/fmt.sh` which runs both `clippy` and `fmt` with the right versions of the rust toolchain via `rustup`
- When writing unit tests, nothing should _ever_ depend on the local environment. This includes hardware detection, existence of pre-installed binaries, internet connectivity, and anything else that could differ between development environments

---
> Source: [ibm-granite-community/granite-cli](https://github.com/ibm-granite-community/granite-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
