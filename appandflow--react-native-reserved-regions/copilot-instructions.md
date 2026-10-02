## react-native-reserved-regions

> This repository publishes `react-native-reserved-regions`, a New Architecture-only

# Agent guide

## Project

This repository publishes `react-native-reserved-regions`, a New Architecture-only
React Native library. `src/` owns the public TypeScript contract and Fabric spec;
`ios/` and `android/` own native observations. `example/` runs React Native 0.88 RC.
`website/` is the Docusaurus documentation site and is never published to npm.

## Development

Use Node 22.18 or newer and the pnpm version pinned in `package.json`.
Read [docs/workflow.md](docs/workflow.md) for the contribution loop and
[RELEASE.md](RELEASE.md) before any release work.

```sh
pnpm install --frozen-lockfile
pnpm run format:check
pnpm run lint
pnpm run typecheck
pnpm test
pnpm run build
pnpm run docs:build
```

Run affected checks while iterating and all defined checks before committing.
Use Oxfmt and Oxlint; do not add ESLint or Prettier alongside them. Keep Bob for
React Native package builds and TypeScript 7 for declarations and type checks.
Keep Jest with React Native's preset for renderer tests.

## Changes

- State the intended behavior and verification before implementation.
- Make the smallest change that satisfies the request. Preserve unrelated work.
- Search upstream issues and pull requests before patching dependency behavior.
- Use `@janic/` branch names when no more specific convention applies.
- Keep public types documented and website examples aligned with the source.
- Test observable behavior: native-to-JS mapping, subscription lifetime, provider
  coordinates and change delivery. Do not test implementation source text.
- A successful unit test or native compile does not prove simulator behavior.
  Record the actual runtime, device and observation; mark unavailable checks.
- Use Stim from `example/` to build/run owned devices, and agent-device for UI
  interactions. Never shut down or modify unrelated devices.
- Preserve zero-width or zero-height divisions. Do not turn every region into
  edge insets or assume every division occludes content.
- iOS APIs introduced in 27.1 use an SDK compile guard and a runtime availability
  check. Older SDKs compile out the feature; older runtimes return no regions.
- Keep native observations attached to the relevant hierarchy/window. Do not
  merge results from unrelated windows into an implicit global value.
- Keep comments only for public API contracts, licenses, required directives,
  or non-obvious external constraints. Name the source of a constraint.

## Pull requests

Follow [docs/workflow.md](docs/workflow.md). Include the behavior, verification,
known limitations and screenshots for visible changes. Explain the final diff,
not the history of revisions. Do not merge or publish unless authorized.

## Releases

Follow [RELEASE.md](RELEASE.md). Publish pnpm-packed tarballs through the Release
workflow using npm trusted publishing. Check the exact tag, manifest, release
notes and package contents. Never force-push release tags or overwrite versions.
Do not report npm permissions, deployment protection or a publish as configured
until verified with the relevant service.

---
> Source: [appandflow/react-native-reserved-regions](https://github.com/appandflow/react-native-reserved-regions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
