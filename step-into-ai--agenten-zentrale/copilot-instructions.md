## agenten-zentrale

> Agentenzentrale is a local-first voice/text cockpit for Hermes agents. A fresh clone must remain useful without provider keys, private media or maintainer-specific paths.

# AGENTS.md — Agentenzentrale operator contract

## Mission

Agentenzentrale is a local-first voice/text cockpit for Hermes agents. A fresh clone must remain useful without provider keys, private media or maintainer-specific paths.

## First commands

```bash
npm ci
npm run setup -- --yes
npm run doctor -- --strict
npm run pack:smoke
npm run verify
```

`npm run bootstrap -- --yes` runs the same install, setup, doctor, pack-smoke and build phases. Do not invent Hermes flags; inspect the local scripts and Hermes CLI before documenting or invoking commands.

## Architecture boundaries

- `app/server.mjs` owns API authorization, SSE, Realtime routing and local state.
- `app/src/` is the React/Vite review and control UI.
- `app/lib/adapters/` owns runtime adapters. Provider credentials never enter generic UI components.
- `app/packs/` contains clone-safe public reference packs.
- `app/data/packs/` and `app/data/agents.local.json` contain private/local packs and overrides; both are ignored.
- Agent visibility, order, persona, permissions, tools and presentation come from manifests.
- New behavior branches on capabilities or declared pack contracts, never on a new product/person identity.
- Legacy compatibility aliases may be reduced but must not be copied into new features.

## Runtime contracts

- Hermes text calls use the local Hermes CLI directly.
- Pack profile/model/provider values are validated before spawning.
- The executable, `cwd` and `HERMES_HOME` are operator-owned boundaries and cannot be redirected by downloaded packs.
- Hermes subprocess output, timeout and cancellation are bounded; stop must terminate the child process.
- Realtime tools are fail-closed. Missing, malformed or unknown declarations authorize nothing.
- OpenAI credentials remain server-side. Browser storage and public config responses must never contain permanent or ephemeral credentials.
- Office mutations retain path checks, confirmation gates and undo behavior.

## Creating an agent

```bash
npm run agent:create -- --id my-agent --name "My Agent" --adapter static
npm run animations:generate -- --agent my-agent
npm run animations:validate -- --agent my-agent
npm run verify
```

See `docs/BUILD_YOUR_AGENT.md` and `docs/ANIMATION_PIPELINE.md`.

## Safety invariants

- Never commit `.env*` except explicit examples, `app/data/`, user documents, voice sessions, generated media or private absolute paths.
- Never bundle third-party names or character art as official assets without rights.
- File tools stay inside configured roots and realpath-check symlinks.
- External sends, destructive writes, installation and service changes require explicit user confirmation.
- Never claim a side effect without a real tool/API result.
- Preserve pre-existing uncommitted changes; do not reset, stash or clean them without explicit direction.

## Test-driven changes

1. Add a failing behavior/contract test.
2. Run it and record the real RED failure.
3. Implement the smallest safe change.
4. Run focused tests and record GREEN.
5. Run the complete suite and production build.

## Definition of done

```bash
npm test
npm run build
npm run verify:release
npm run release:stage
npm run clean-room
```

For release work, inspect the staged artifact rather than the live checkout. No push or tag is implied by a successful local release gate.

---
> Source: [step-into-ai/Agenten_zentrale](https://github.com/step-into-ai/Agenten_zentrale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
