## gpt-workflow

> Development requires Bun 1.3 or newer. Node.js is not a supported runtime,

# Working on gpt-workflow

Development requires Bun 1.3 or newer. Node.js is not a supported runtime,
for development or otherwise.

## Verify changes

```sh
just check
```

`just check` runs formatting checks (`ultracite`), offline verification,
package packing and installation, strict consumer typechecking, and installed
CLI smokes. It does not spend model tokens and is the gate for every change.

`just verify` runs the live App Server suite and does spend model tokens —
run it only when a change touches live agent behavior and you have an
authenticated Codex CLI.

`just fmt` applies formatting fixes.

## Reference workflows and mirroring

The executable reference workflows live in `.claude/workflows/` and are
mechanically mirrored into `.codex/workflows/` for the test suite. Edit the
`.claude/workflows/` originals, then regenerate the fixtures:

```sh
just mirror
```

Never hand-edit the mirrored copies in `.codex/workflows/`.

The Claude material used as parity reference is preserved under
`.claude/workflows/docs/` and is not part of the Codex package contract; the
parity ledger lives at `docs/08-claude-parity.md`.

## Documentation rules

`docs/00-audience.md` is the contract for all user-facing docs, including
`README.md`: who the docs serve, what pages may assume, journal terminology,
copy-runnable examples, in-section failure semantics, and token-spend
labeling. `README.md` is written for someone discovering the project for the
first time; developer instructions belong here, not there.

---
> Source: [CyrusNuevoDia/gpt-workflow](https://github.com/CyrusNuevoDia/gpt-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
