## vapi-network

> This repository publishes one user-facing distribution, `vapi-network`, and

# Repository guide

This repository publishes one user-facing distribution, `vapi-network`, and
four scoped building blocks. Keep the implementation small, testable, and
non-custodial.

## Layout

- `packages/core`: protocol primitives, shared types, wallet, spend policy,
  network guard, discovery merge, and local receipt ledger
- `packages/sources`: discovery-source adapters
- `packages/mcp`: stdio MCP server and tool registration
- `packages/cli`: command parsing and command implementations
- `packages/vapi-network`: bundled public distribution and `vapi` binary
- `examples`: direct SDK usage, type-checked by `pnpm typecheck` through
  `examples/tsconfig.json`
- `scripts`: repository-wide staged publishing and tarball verification

Dependencies point inward: `core` has no workspace dependency; `sources`
depends on core; interface packages may depend on core and sources. Do not make
core aware of the CLI, MCP protocol, or a specific registry.

## Commands

Use Node.js 22 and pnpm 10.

```sh
pnpm install
pnpm format:check
pnpm lint
pnpm typecheck
pnpm test
pnpm build
pnpm pack:check
```

Run the full relevant checks before handing work off. Tests must be deterministic
and must not require a funded wallet or a live network. Prefer injected clocks,
randomness, filesystem roots, and `fetch` implementations.

## Invariants

- The project is non-custodial. Private keys and passphrases never leave the
  machine. Never introduce a payment proxy or server-side signer into the
  default flow.
- Apply spend policy before signing. A failed policy check must not mutate spend
  state or create a payment authorization.
- No telemetry leaves the machine by default. Any future exporter requires an
  explicit user opt-in and must never include secrets.
- The configuration home is `~/.vapi/`, overridable with `VAPI_HOME`. Its stable
  entries are `config.json`, `wallets.json`, `wallets/<name>.json`,
  `wallets/.trash/`, `audit.log`, `receipts.jsonl`, `searches.jsonl`, and
  `spend-ledger.json`. `keystore.json` is a compatibility symlink left by the
  0.2.x migration and is kept for one release.
- Migration from `~/.vapi/agent-cash/` copies files, never deletes the old
  directory, never overwrites new state, and prints a notice.
- Receipts are an append-only JSONL ledger. Spend accounting uses
  `spend-ledger.json`; keep crash-safe writes and concurrent access in mind.
- Discovery de-duplicates on normalized resource URL without discarding
  provenance.
- MCP tools use product namespaces. Deprecated aliases must identify themselves
  with a one-line deprecation notice and may not silently diverge in behavior.
  Every registered tool name is named in the root README; `tool-docs.test.ts`
  enforces it, as `help-alignment.test.ts` does for every CLI command.
- Protect every outbound service request with the network guard. Treat redirects
  and resolved IP addresses as new destinations that need validation.
- Published package versions move together. All packages are currently
  `0.3.0`, which `scripts/pack-check.mjs` and `packages/cli/src/version.ts`
  pin as well. Every package's `publish:npm` goes to the `latest` npm tag, and
  every package's `publish:npm:next` goes to `next`. Access is always public.
- Every published tarball has zero runtime dependencies. All runtime code is
  bundled with esbuild; builds must fail if non-Node external imports leak into
  `dist`.
- Do not commit generated `dist/`, `publish/`, coverage, or tarball artifacts.
- Preserve the Apache-2.0 license and provenance comments on extracted contract
  schemas.

## Package and API work

TypeScript is strict ESM. Use explicit `.js` extensions for relative imports
that survive in emitted JavaScript. Prefer small public surfaces and keep
network, filesystem, and time dependencies injectable where behavior needs a
unit test.

`Source`, `Signer`, `Policy`, `Receipt`, and `Listing` are core contracts.
Source adapters tolerate incomplete third-party data at their boundary and
return normalized listings; callers should not need source-specific branches.

CLI commands support `--json`, including errors, and return documented exit
codes. Machine-readable output goes to stdout; human diagnostics and migration
or deprecation notices go to stderr when they must not corrupt JSON.

## Publishing

`pnpm pack:check` first builds the whole workspace, then
`scripts/prepare-publish.mjs` stages every non-private `packages/*` package into
its own `publish/` directory. npm reads `package.json` before lifecycle hooks,
so maintainers publish only these staged directories.

Package manifests must list literal top-level paths in `files`; the staging
script intentionally rejects globs. The staged manifest removes scripts and all
dependency sections, and the verifier scans every staged file for `workspace:`
before asking npm for a dry-run tarball report. Never weaken these checks to
make a release pass.

Release publishing is manual and must use `--access public` with an explicit
tag: `publish:npm` sends every package — the `vapi-network` distribution and the
four scoped packages — to `latest`, and `publish:npm:next` puts the same package
on `next` instead. The manual GitHub workflow verifies artifacts and prints
commands; it has no npm authentication and performs no publish.

---
> Source: [vAPI-Network/vapi-network](https://github.com/vAPI-Network/vapi-network) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
