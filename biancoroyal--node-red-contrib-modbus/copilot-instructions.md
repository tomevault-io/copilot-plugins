## node-red-contrib-modbus

> | | **This repo (v5 OSS)** | **v6 closed** |

# CLAUDE.md — node-red-contrib-modbus v5 (Open Source)

## Repository Context

| | **This repo (v5 OSS)** | **v6 closed** |
|---|------------------------|-----------------|
| Path | `BIANCO-ROYAL/node-red-contrib-modbus` | `P4NR/modbus/node-red-contrib-modbus` |
| Package | `node-red-contrib-modbus` | `@plus4nodered/node-red-contrib-modbus` |
| Version | `5.x` LTS | `6.0.0-beta.x` |
| License | BSD-3-Clause, public GitHub | `private: true` |
| TLS | Not in v5 | `Modbus-Client-TLS` in v6 |
| Server | `Modbus-Server` in this package | Separate contrib-modbus-server pkg |
| Logging | `debug` (`DEBUG=contribModbus*`) | `winston` in v6 |

**Use this repo** for public OSS work, community PRs, and v5 LTS maintenance.
**Use v6** for TLS and next-major architecture — not for backporting without explicit decision.

## Architecture

- Source: `src/` → built to `modbus/` via `npm run build` (Babel/Gulp)
- Core: `src/core/`
- Nodes: client, read/write, flex, **server**, queue-info, response, etc.
- FSM: `@xstate/fsm` for connection state
- Modbus lib: `@openp4nr/modbus-serial` (Cloudsmith)
- Server: `jsmodbus`

## Commands

```bash
npm run build
npm run lint
npm test
npm run test:units
npm run test:e2e
npm run mocha:base -- test/units/modbus-client-test.js
```

## Testing

- **Mocha** + `node-red-node-test-helper` (not Jest for node tests)
- Unit: `test/units/`, E2E: `test/e2e/`, flows in `test/*/flows/`
- Dynamic ports: `test/helper/test-helper-extensions.js`
- Always `npm run build` after `src/` changes

## OSS Contribution Rules

- Public API stability matters — avoid breaking changes in v5 without major bump
- CHANGELOG entry for user-visible changes
- Conventional commits; PR-ready diffs
- No secrets in code or PR descriptions

## P4NR Agent Team (Cursor)

See `docs/p4nr/README.md`.

| Team | Agent | Role |
|------|-------|------|
| 1 | `p4nr-spec-author` | Specs + plans |
| 2 | `p4nr-spec-reviewer` | APPROVE/REJECT |
| 3 | `p4nr-developer` | TDD in `src/` + `test/` |

Flow: Team 1 → Team 2 → GATE 1 → Team 3 → GATE 2.

---
> Source: [BiancoRoyal/node-red-contrib-modbus](https://github.com/BiancoRoyal/node-red-contrib-modbus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
