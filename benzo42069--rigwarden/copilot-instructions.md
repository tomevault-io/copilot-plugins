## rigwarden

> These instructions apply to every agent working in this repository.

# AGENTS.md — RigWarden

These instructions apply to every agent working in this repository.

## Mission

Build **RigWarden**, *an open editor for modern modelers*: a free, local-first, accessible, cross-platform editor beginning with Fractal Audio hardware.

## First actions

Before editing production code:

1. Read `docs/00-READING-ORDER.md` and every required file.
2. Read `.codex/skills/enforce-topology-strict-tdd/SKILL.md` and its references.
3. Inspect `work-items/index.yaml`.
4. Confirm the selected packet is `READY`, all dependencies are `INTEGRATED`, and paths still match.
5. Confirm the test/hardware/fixture environment can execute the required cycle.
6. If you are the parent orchestrator, verify effective child model routing before a large fan-out.

## Binding rules

- Strict observed RED–GREEN–REFACTOR is mandatory for production behavior.
- No production behavior before the focused RED is run and confirmed.
- No protocol guessing.
- No unsupported compatibility claims.
- No copied vendor/competitor artwork, distinctive layouts, binaries, or unknown-rights fixtures.
- No Node runtime, local HTTP service, or WebView editor shell in production mobile builds.
- Flutter owns presentation; Rust owns deterministic domain/protocol/state behavior; Swift/Kotlin adapters remain narrow.
- No account, mandatory cloud, or project AI backend.
- AI never receives raw transport access.
- Unknown firmware is read-only or unsupported.
- Accessibility equivalent ships with each visual workflow.
- All production icons/decorative controls are PNG; only knobs may be procedurally rendered. Dynamic live-data graphics are permitted.
- No placeholder control or screen counts as implemented.
- A simulator pass is not hardware verification.
- A fake platform adapter is not native-platform verification.
- A semantics-tree pass is not VoiceOver/TalkBack physical verification.
- Do not edit shared integration-owned files unless the packet explicitly grants ownership.
- Do not broaden a work packet while implementing it.

## Work packet discipline

An implementation agent executes exactly one or a small set of tightly related leaf packets.

The packet must include:

- immutable IDs;
- dependencies;
- one observable behavior;
- non-goals;
- allowed/forbidden paths;
- exact test command;
- expected RED;
- minimum GREEN;
- required sweeps;
- evidence directory;
- acceptance and block conditions.

When a packet is wrong or stale, stop and return it to the parent for amendment. Do not silently reinterpret it.

## Subagent roles

- `topology_explorer`: read-only mapping.
- `topology_fixture_researcher`: read-only source/provenance research.
- `topology_implementer`: strict-TDD implementation.
- `topology_reviewer`: independent correctness/evidence review.
- `topology_accessibility_reviewer`: nonvisual/semantics review.
- `topology_security_reviewer`: security/trust-boundary review.
- `topology_test_runner`: commands/evidence only; no source edits.
- `topology_release_verifier`: clean-clone/release verification.

Implementers do not self-approve. The parent or integration owner lands patches and reruns required sweeps.

## Shared files

Unless a packet explicitly says otherwise, only the integration owner modifies:

- root `Cargo.toml`;
- lockfiles;
- root `pubspec.yaml`/workspace package lists;
- generated FFI registries;
- device-pack indexes;
- requirement traceability matrix;
- compatibility report inputs;
- release manifests;
- global changelog;
- global license/notice manifests.

Workers put proposed shared-file changes in `.tdd/evidence/<id>/handoff.md`.

## Status truth

Use only the project statuses defined in the strict-TDD skill. Source code present in a worker branch is not integrated. A skipped test is not verified. A planned design is not implemented.

## Autonomy

When one packet is blocked, record the blocker and continue independent ready work. Ask the user only for actions that genuinely require them: physical hardware manipulation, credentials/signing, legal/product decisions that cannot be inferred, or inaccessible proprietary inputs.

Never promise later/background work. Complete as much as the current session and tools allow, then report exact integrated state.

---
> Source: [benzo42069/rigwarden](https://github.com/benzo42069/rigwarden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
