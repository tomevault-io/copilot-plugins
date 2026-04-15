## michael-simoneau-com

> - Build and runtime are Expo-first with Expo Router + Metro; do not reintroduce Vite, CRA, or other legacy build systems.

# GEMINI.md

## Project Defaults
- Build and runtime are Expo-first with Expo Router + Metro; do not reintroduce Vite, CRA, or other legacy build systems.
- Use Yarn for dependency and script management; do not add `package-lock.json`.
- Primary development scripts are `yarn start` for local development and `yarn web` for web preview.

## Coding Standards
- Keep TypeScript strict and prefer explicit types, discriminated unions, and descriptive interfaces.
- Use React Native primitives for UI (`View`, `Text`, `ScrollView`, etc.); avoid direct DOM-specific APIs in runtime code.
- If a web-only capability is required, gate it behind runtime checks that respect `useFoundationRuntime()`.
- Build feature logic under `src/features`; promote reusable UI into `src/ui`; keep global provider composition in `src/providers`.
- Use `src/foundation` primitives for config, analytics, and runtime intelligence in cross-cutting concerns.
- Keep foundation runtime detection platform-safe; avoid direct `window` or `document` access in shared runtime logic.
- Keep analytics sinks typed and side-effect free by default.
- Favor serializable content pipelines compatible with Expo/React Native assets; avoid Node-only runtime dependencies.
- Model service classes with a singleton-oriented pattern and keep explicit error handling, type safety, documentation, and tests.
- Model functions with the repository `onFlow` pattern and include validation, auth checks, error handling, and documentation.

## Verification
- Run `yarn typecheck` before completion and before commit.
- Add or update tests for changed behavior, favoring React Native Testing Library for component validation.
- Add smoke coverage for critical navigation flows when shipping substantial UX changes.
- For service and function changes, preserve error handling, type safety, and test coverage expectations from repository rules.
- If changes touch `src`, `src/services`, `functions`, or `functions/src/__tests__`, ensure required `README` guidance exists with architecture, implementation, performance, security, testing, and documentation sections.

## Safety Rules
- Do not run destructive Git commands (for example `git reset --hard`) unless explicitly requested.
- Do not revert unrelated local changes you did not author.
- Never commit secrets, credentials, or environment-sensitive values.
- Keep guidance and changes forward-compatible with the Expo/React Native Web architecture.

## Update Policy
- Keep this file concise and operational; include durable implementation rules, not temporary task notes.
- When `AGENTS.md` and this file overlap, keep repository-specific constraints and remove duplicate wording.
- Update this file when architectural, tooling, or verification expectations materially change.

## Learned Knowledge References
- Use `docs/learned/dotcom-learned-knowledge.md` as the canonical durable-facts aggregate.
- Use `docs/learned/dotcom-learned-decisions.md` for recurring command/workflow preferences.
- Use `docs/learned/dotcom-learned-evidence-index.md` to trace claims back to parent transcript evidence.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelSimoneau)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MichaelSimoneau)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
