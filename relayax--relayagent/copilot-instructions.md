## relayagent

> Guidance for coding agents working **on the RelayAgent substrate itself** (this repository).

# CLAUDE.md

Guidance for coding agents working **on the RelayAgent substrate itself** (this repository).
If you are looking for how to *use* RelayAgent, read [README.md](README.md).

## Premise

Everything can be expressed as an agent package. The substrate's job is to judge packages,
not to be clever about them.

## Layout

```
runner/                 the substrate (CLI + daemon). Plain .ts, run with --experimental-strip-types.
  relay.ts              CLI entry and command dispatch
  manifest.ts           relay.yaml parsing + judgment against relay.manifest.yaml
  installer.ts          install: judge, build surfaces, record grants in the ledger
  api.ts                daemon HTTP API + console hosting
  session.ts, run.ts    harness sessions
  draft.ts              edit layer (git-backed drafts) and publish gate
  scripts.ts            script (verb) execution
  vault.ts              credentials: macOS Keychain, 0600 file fallback
  state.ts, tick.ts     ledger, logs, triggers
  conform.ts            harness/channel contract conformance
  build.ts, login.ts

relay.manifest.yaml     the grammar: JSON Schema for relay.yaml, with commentary
relay.yaml              a full worked example manifest
packages/system         the management shell, itself a package (console, studio, harness adapters)
packages/*              reference packages
lib/relayjs             browser-side client used by package views
assets/                 project logo
```

## Rules that are not negotiable

1. **The manifest is the BOM.** `relay.yaml` owns structure and paths; the tree owns content.
   Never make the runner read a path that the manifest does not declare.
2. **Fail-loud.** A mismatch between declaration and reality fails validation or install.
   Do not add warnings, fallbacks, or silent degradation. There are no warnings, only judgments.
3. **Declarations are caps, grants are approvals.** A grant recorded in the ledger can never
   exceed what the manifest declared. Never widen a grant at runtime.
4. **Credentials never live in the tree.** Manifests declare only the shape of auth
   (`none`, `token`, `oauth`). Values go through `runner/vault.ts`. Never write a secret to
   a manifest, a fixture, a log line, or a test.
5. **Harness-neutral agents.** Agents ship as a neutral bundle (persona, skills, commands, meta).
   Translation into a native CLI format belongs to the adapter in `packages/*/harness/<name>/`,
   never to the runner. Do not hardcode a model name, a vendor alias, or a CLI flag in `runner/`.
6. **Minimal ground.** A session stands on one granted workspace folder. Extra folders are `dir`
   services. `~/.relay` is denied to every session, always.

## Working on the runner

- Target Node.js 22.6+. The runner is TypeScript executed directly via
  `node --experimental-strip-types`. That means **type-only syntax only**: no enums, no
  namespaces, no decorators, no parameter properties, nothing that needs emit.
- No build step and no runtime dependency beyond `yaml`. Do not add dependencies to the root
  `package.json` without a strong reason; a new dependency is a new thing every user must trust.
- Keep the CLI surface in `runner/relay.ts` a thin dispatcher. Logic belongs in the module.

## Changing the grammar

Any change to what `relay.yaml` may contain touches four places at once. Change all four in the
same commit or the change is incomplete:

1. `relay.manifest.yaml` (the schema and its commentary)
2. `runner/manifest.ts` (parsing and judgment)
3. `relay.yaml` (the worked example) and any affected `packages/*/relay.yaml`
4. The README tables, in **all four languages** (`README.md`, `.ko`, `.zh-CN`, `.ja`)

## Verify before you claim done

```sh
node --experimental-strip-types runner/relay.ts validate packages/system
node --experimental-strip-types runner/relay.ts validate packages/scribe
node --experimental-strip-types runner/relay.ts validate packages/video-studio
node --experimental-strip-types runner/relay.ts validate packages/music-sommelier
npx tsc --noEmit
```

All four packages must print a passing judgment. CI runs exactly this.

## Language

Code comments and manifest commentary in this repo are written in Korean; public-facing docs
(READMEs, CONTRIBUTING, issue templates) are English-first with `ko` / `zh-CN` / `ja` translations.
Follow whichever convention the file you are editing already uses. Do not mass-translate a file
as a side effect of an unrelated change.

## Never

- Never commit a build artifact (`*.tsbuildinfo`, `surfaces/*/out`, `node_modules`).
- Never commit an absolute local path, a machine name, or a personal directory.
- Never commit anything under `~/.relay` (ledger, vault, sessions, logs) as a fixture.

---
> Source: [relayax/relayagent](https://github.com/relayax/relayagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
