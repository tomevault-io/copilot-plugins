## workspace

> - [`README.md`](README.md) — what this repo is and how the pieces

# Agent guidelines

## Read these first

- [`README.md`](README.md) — what this repo is and how the pieces
  fit together.
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — setup, checks, commit and
  pull request conventions. The canonical source for the day-to-day
  workflow.
- [`docs/README.md`](docs/README.md) — design specification. Forward-
  looking; treat as intent, not as a description of `main`.
- [`docs/08_capnweb_interface.md`](docs/08_capnweb_interface.md) —
  the RPC contract between the Durable Object and `wsd`. Required
  reading before touching `packages/rpc` or `packages/workspace`.
- Each package's own `README.md` — implementation status and
  package-specific notes.

## Skills

In-repo skills live under [`.agents/skills/`](.agents/skills/). Load
the file directly when the trigger applies:

| Skill | Load when |
|---|---|
| [`prose`](.agents/skills/prose/SKILL.md) | Writing code comments, commit messages, READMEs, or documentation. |
| [`pull-requests`](.agents/skills/pull-requests/SKILL.md) | Writing or editing a pull request description. |
| [`test-driven-development`](.agents/skills/test-driven-development/SKILL.md) | Implementing logic, fixing a bug, or changing behavior. |
| [`capnweb`](.agents/skills/capnweb/SKILL.md) | Touching anything that crosses the RPC boundary: `packages/rpc`, `packages/workspace`, the `wsd` client, or the Durable Object server. |
| [`cloudflare`](.agents/skills/cloudflare/SKILL.md) | Index of host-side Cloudflare skills — Workers, Durable Objects, wrangler, sandbox SDK, agents SDK. |

## Checks before you finish

Run from the repo root:

```bash
npm run format        # biome format --write .
npx biome check .     # biome lint + formatter verification
```

`biome check` must exit zero. Fix the underlying issue rather than
silencing the rule.

Then run the package-level tests for whatever you touched:

```bash
npm test                                                  # whole workspace
npm test --workspace @cloudflare/dofs                     # one package
npm test --workspace @cloudflare/dofs -- src/foo.test.ts  # one file
```

Full details, including typecheck and build commands, are in
[`CONTRIBUTING.md`](CONTRIBUTING.md).

## Commits and pull requests

Follow [`CONTRIBUTING.md`](CONTRIBUTING.md). The short version:

- One logical change per commit.
- Imperative subject prefixed with the scope (`dofs:`, `rpc:`, `wsd:`,
  `workspace:`, `examples/think:`, `docs:`, `ci:`). Multiple scopes
  joined with commas.
- Self-contained body wrapped at 72 characters. No references to chat
  history, agent sessions, sibling commit SHAs, or task identifiers.
- No emojis. No marketing voice. Prose paragraphs in the body, not
  bulleted lists.

Full guidance is in [`.agents/skills/prose/SKILL.md`](.agents/skills/prose/SKILL.md)
and [`.agents/skills/pull-requests/SKILL.md`](.agents/skills/pull-requests/SKILL.md).

## Scripts

Two top-level directories hold helper scripts. They're outside the
workspace package set on purpose — they're tooling, not shipped code.

[`scripts/`](scripts/) holds repo-maintenance scripts run from npm:

- `set-versions.mjs` syncs the version of every published package in
  lockstep. Invoked from the release pipeline.

[`script/`](script/) holds operator-facing harnesses for `wsd` and
the sync loop. Reach for these when you're chasing a behavior the
unit tests don't cover.

- `shell` boots a debian-slim container with the linux `wsd` binary
  mounted under `/usr/local/bin`. The starting point for anything
  that needs a real FUSE mount.
- `wsd-soak.mjs` boots two `wsd` containers wired peer-to-peer and
  soaks the sync loop. Use it to chase convergence or churn bugs.
- `wsd-stub-soak.mjs` soaks the long-lived WebSocket session and
  reads `session.getStats()` to detect stub-disposal drift. Run it
  for changes around the capnweb lifecycle.
- `wsd-fuse-flush.mjs` end-to-end checks that the FUSE driver spills
  its in-memory write buffer into the backing VFS, so a capnweb-side
  `pullOnce` actually sees the bytes.
- `fs-tests.sh` / `run-fs-tests.sh` run the filesystem conformance
  harness against the FUSE mount.
- `fs-bench.sh` / `run-fs-bench.sh` benchmark common development
  tasks against the mount with a tmpfs baseline for comparison.
- `exec-tests` boots `wsd` in docker with FUSE disabled and exercises
  a few `shell.exec` scenarios.

When you add a script, drop a one-line description at the top of the
file and add it to the list above.

## Project-specific patterns

- **RPC.** The wire contract is capnweb. Read
  [`docs/08_capnweb_interface.md`](docs/08_capnweb_interface.md) and
  the [`capnweb`](.agents/skills/capnweb/SKILL.md) skill before
  changing anything that crosses the Durable Object ↔ `wsd` boundary.
  Stubs are object-capabilities; dispose them. Leaks are tracked by
  the harness in `packages/rpc`.
- **Storage.** `packages/dofs` is the authoritative SQLite layer.
  Filesystem primitives operate on a `Database` handle; the sync
  protocol building blocks share the same handle. The package README
  enumerates the exported surface.
- **FUSE shim.** `packages/wsd` runs in the sandbox container. FUSE-
  backed tests only run on Linux and are skipped elsewhere
  automatically.
- **Examples are real consumers.** `examples/think` and
  `examples/wsd-container` exercise the public surface. If you change
  a public API, update the examples in the same change.

---
> Source: [cloudflare/workspace](https://github.com/cloudflare/workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-08 -->
