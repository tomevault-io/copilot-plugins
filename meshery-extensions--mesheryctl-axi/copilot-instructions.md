## mesheryctl-axi

> `mesheryctl-axi` is a Node/TypeScript [AXI](https://axi.md/) wrapper that spawns the

# Project agent memory

`mesheryctl-axi` is a Node/TypeScript [AXI](https://axi.md/) wrapper that spawns the
`mesheryctl` binary and reshapes its output for agents. Design and v1 scope:
[meshery/meshery#20979](https://github.com/meshery/meshery/issues/20979).

## Working here

- Build/test: `make setup build tests` (see [`Makefile`](Makefile)); CI is
  [`.github/workflows/node-checks.yml`](.github/workflows/node-checks.yml) on Node 22 and 24.
- Most other files under `.github/workflows/` are leftovers from `meshery/repo-template`
  (Go server, UI, Helm, docs) pending removal in
  [#2](https://github.com/meshery-extensions/mesheryctl-axi/issues/2). Their runs are not a signal
  for this package; the ones that failed on every PR are disabled in Settings → Actions.
- Product-code changes validate through the `no-mistakes` pipeline (`no-mistakes` remote).
- Releases are automation-driven. Never `npm publish`, `npm version`, or tag by hand. Follow
  [`docs/release-procedure.md`](docs/release-procedure.md) /
  [`.agents/skills/mesheryctl-axi-release`](.agents/skills/mesheryctl-axi-release/SKILL.md).

## Output contract (do not break without a `minor` label - pre-1.0)

- Reporting (list/view/status/errors) is TOON; design/model **content** is raw YAML/JSON, never TOON.
- Errors are `AxiError` with a code from [`src/errors.ts`](src/errors.ts); unknown flags are
  rejected per subcommand via the `*_FLAGS` tables in `src/commands/`.
- Every successful output ends with `help[]` from [`src/suggestions.ts`](src/suggestions.ts);
  empty collections render the definitive `<label>: 0`.

## Sharp edges

- The unit tests mock `mesheryctl` through `setMesheryctlRunner`, so they cannot catch a flag or
  subcommand that real `mesheryctl` rejects. Before passing a new argument, check
  `mesheryctl <command> --help` on a current release. As of v1.0.69, `list` commands and
  `system status` / `system context view` accept no `--output-format`; only `view` commands do
  ([#5](https://github.com/meshery-extensions/mesheryctl-axi/issues/5)).

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.

---
> Source: [meshery-extensions/mesheryctl-axi](https://github.com/meshery-extensions/mesheryctl-axi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
