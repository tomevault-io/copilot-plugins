## gui-shell-public

> GUI-Shell is a Windows-first desktop GUI/runtime/evidence shell for AI-assisted development workflows.

# AGENTS.md

## Project Identity

GUI-Shell is a Windows-first desktop GUI/runtime/evidence shell for AI-assisted development workflows.

This repository is designed to be readable and safely modifiable by LLM-based development agents, including Codex-like coding agents. The repository exposes source code, validation tooling, schemas, release gates, evidence boundaries, and public review material.

No OpenAI endorsement is claimed. Product release readiness is not claimed.

## Agent Mission

Agents may help inspect, validate, document, and safely extend this repository.

Agents must preserve:

- authority boundaries
- release-gate integrity
- evidence provenance
- fail-closed behavior
- public/private boundary hygiene
- Windows-first v1.0 scope

## Authority Hierarchy

When instructions conflict, follow this order:

1. Explicit owner/user instruction, unless it weakens safety, authority, evidence, release-gate, owner GO, `release_ready`, audit, recovery, public/private boundary, or content-exposure boundaries
2. This `AGENTS.md`
3. Repository documentation
4. Source code and tests
5. Existing style and inferred conventions

Do not override explicit safety, authority, evidence, or release boundaries based on inference.

## Hard Prohibitions

Agents must not:

- set `release_ready=True`
- record owner GO
- claim OpenAI endorsement
- claim product release readiness without explicit owner approval
- fabricate or synthesize evidence
- manually edit release blockers to force a pass
- promote CONFIG, FIXTURE, mock, or development evidence into release evidence
- bypass policy, invariant, release gate, or evidence checks
- commit secrets, tokens, local paths, private logs, or machine-specific data
- use public promotional assets as canonical release evidence
- alter authority cutover or command dispatch without explicit instruction

## Public Scope

This public repository may include:

- source code
- validation tools
- schema and conformance tests
- public documentation
- redacted Windows proof assets
- OpenAI/Codex application support materials

This repository must not include:

- unredacted private logs
- raw local-only release evidence
- secrets or credentials
- owner-private decision logs
- machine-specific transcripts
- private repository-only notes

## Safe Edit Zones

Agents may edit these areas for ordinary public-safe tasks:

- `README.md`
- `QUICKSTART.md`
- `docs/public/`
- `docs/application/`
- `docs/agents/`
- `examples/`
- non-authority UI polish in `apps/desktop_flutter/`
- tests and validation additions
- redacted public assets

## Restricted Edit Zones

These files and directories require extra caution and explicit rationale:

- `tooling/release_gate_check.py`
- `tooling/windows_release_evidence.py`
- `tooling/evidence_bundle.py`
- `tooling/release_runtime_assertions.py`
- `release_blockers.registry.json`
- `installer/windows/`
- `native/rust_helper/`
- `packages/shell_core/`
- `MANIFEST.sha256.json`

When editing restricted zones, explain:

- why the change is necessary
- which boundary is affected
- which validation commands were run
- whether release behavior changed

## No-Touch Unless Explicitly Instructed

Do not modify these without explicit user instruction:

- owner GO flow
- `release_ready` flow
- release evidence promotion logic
- authority cutover
- command dispatch enablement
- credential or secret handling
- private evidence handling

## Required Validation

After any non-trivial change, run the available subset of:

```bash
python tooling/schema_check/check_schemas.py
python tooling/conformance_tests/run_conformance_skeleton.py
python tooling/manifest.py --check
python tooling/release_gate_check.py
python tooling/evidence_bundle.py --check
python tooling/release_runtime_assertions.py --check
python tooling/validate_all.py --python-only
```

If the host uses `python3`, use the same commands with `python3`.

If Rust tooling is available:

```bash
cd native/rust_helper
cargo fmt --check
cargo test
cd ../..
```

If Flutter tooling is available:

```bash
cd apps/desktop_flutter
flutter analyze
flutter test
dart format --output=none --set-exit-if-changed .
cd ../..
```

## Release Gate Expectations

Normal validation should pass.

Strict release may fail if owner GO has not been recorded. Agents must report this state honestly. Do not force strict release to pass by editing blockers, evidence, or owner approval fields.

Expected safe state before owner approval:

- `release_ready` remains false or unset
- owner GO is absent
- product release readiness is not claimed

## Evidence Rules

Evidence must be real, traceable, and correctly classified.

Agents must not:

- convert promotional screenshots into release evidence
- convert local dev-tree output into installed-path evidence
- remove blockers without validator support
- hand-edit evidence to satisfy a gate

Public proof assets are reference and promotional materials. Canonical release evidence must remain governed by the release tooling.

## Final Response Format

When completing a task, report:

1. changed files
2. commands run
3. validation results
4. release gate status
5. release blockers that remain active
6. whether `release_ready` changed
7. whether owner GO changed
8. whether public/private boundary issues were found
9. whether any evidence files were created, edited, or copied

---
> Source: [gatchimuchio/GUI-Shell-Public](https://github.com/gatchimuchio/GUI-Shell-Public) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
