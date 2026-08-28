## ytsaurus-rust-sdk

> - `SPEC/` owns the desired public contract.

# Repository rails

## Sources of truth

- `SPEC/` owns the desired public contract.
- Root `README.md` owns navigation and verified current status.
- `AGENTS.md` owns repository workflow rules.
- Tests and Git history own implementation evidence.
- Goals and `update_plan` own execution plans.
- Do not create `plans/`, `docs/`, or feature-specific Markdown files.
- A colocated README is allowed only for copied third-party source provenance.

## Documentation

- Search existing Markdown before editing documentation.
- Update the nearest canonical file instead of creating another file.
- Create a Markdown file only when the user explicitly requests that artifact.
- Link to canonical text instead of copying it.
- Delete stale duplicates in the same change that replaces them.
- Keep desired behavior in `SPEC/` and verified status in root `README.md`.
- Do not create narrative test receipts. Keep assertions executable.
- Do not mix roadmap status with API contracts.

## Evidence

- `PASS` means the exact named command ran successfully on the current tree.
- An ignored test is not passing evidence.
- A file under `examples/` is not working evidence by itself.
- `ported` means the example is a registered Cargo target and compiles.
- Report every excluded example target beside a green aggregate gate.
- Do not use target omission or `autoexamples = false` as proof of compatibility.
- A Docker E2E claim requires a live run against the pinned image.
- Keep proxy evidence, local evidence, and cluster evidence distinct.

## Changes

- Add one responsibility per commit.
- Keep checkpoint commits reachable from the active branch.
- Run focused RED and GREEN checks before the full workspace gate.
- Run `git diff --check` before every commit.
- Keep unrelated user changes untouched.

---
> Source: [AntonProkopyev/ytsaurus-rust-sdk](https://github.com/AntonProkopyev/ytsaurus-rust-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
