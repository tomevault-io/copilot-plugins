## react-native-plain-text

> A React Native library exposing `PlainText`: a lightweight static-text component

# AGENTS.md

A React Native library exposing `PlainText`: a lightweight static-text component
backed directly by the platform's native text widget (`UILabel` on iOS,
`TextView` on Android) instead of RN's own `<Text>`. It is a **Fabric (New
Architecture) native component**.

**Yarn v4 workspaces — never use `npm`.** The library is the repo root; `example/`
is an Expo dev-client app for running and testing changes.

## Commands

Run from the repo root:

|                                             |                                                  |
| ------------------------------------------- | ------------------------------------------------ |
| `yarn validate`                             | All checks: lint, format, typecheck, test, build |
| `yarn typecheck`                            | TypeScript                                       |
| `yarn lint`                                 | ESLint + Prettier (`--fix` to autofix)           |
| `yarn format`                               | Prettier on non-JS files too (md, json, yml)     |
| `yarn test`                                 | Jest (`yarn test path -t "name"` for one case)   |
| `yarn example ios` / `yarn example android` | Build & run the example app                      |
| `yarn example start`                        | Metro only — does **not** rebuild native         |
| `yarn prepare`                              | Build the shippable library into `lib/`          |

**After every change run `yarn validate`.**

**Do not build the native binaries yourself** (`yarn example ios|android`) unless
explicitly asked to.

## Documentation

Agent docs live in `docs/agent/` — put new ones there and link them here. The
human-facing docs are `README.md` and `CONTRIBUTING.md`, under `docs/` once that
exists.

| Read when                                                                     |                                                                                                                             |
| ----------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Adding or changing any prop                                                   | [architecture.md](docs/agent/architecture.md) — the four-layer prop flow, naming, prop conventions, example app, RN sources |
| **Before** adding a size-affecting prop                                       | [sync-points.md](docs/agent/sync-points.md) — the files that must change together and that nothing verifies                 |
| Implementing a feature                                                        | [workflow.md](docs/agent/workflow.md) — parity policy, order of work, build policy                                          |
| Touching measurement, shadow nodes, JNI or CMake                              | [intrinsic-sizing.md](docs/agent/intrinsic-sizing.md) — how self-sizing works on each platform                              |
| A native change "does nothing", or builds misbehave                           | [native-gotchas.md](docs/agent/native-gotchas.md)                                                                           |
| Quoting or producing a performance number                                     | [measuring.md](docs/agent/measuring.md) — metrics and procedure                                                             |
| Optimizing anything                                                           | [performance.md](docs/agent/performance.md) — what's been done, what was rejected and why, and what's still open            |
| Comparing `PlainText` to RN's `<Text>`, or explaining why this library exists | [rn-text-history.md](docs/agent/rn-text-history.md) — how RN core measures text, and why it never used `UILabel`            |

---
> Source: [mdjastrzebski/react-native-plain-text](https://github.com/mdjastrzebski/react-native-plain-text) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
