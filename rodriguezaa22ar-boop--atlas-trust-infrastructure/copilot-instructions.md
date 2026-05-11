## atlas-trust-infrastructure

> This repository contains `atlas-lab-toolkit`, a local-first, shell-native toolkit for authorized security assessment workflows.

# AGENTS.md

## Project

This repository contains `atlas-lab-toolkit`, a local-first, shell-native toolkit for authorized security assessment workflows.

Atlas is the operator control plane. It coordinates scope, targets, recon, evidence, findings, validation, reports, retention, and release-trust artifacts without becoming a monolith or replacing the underlying domain tools.

Primary domain split:

- `atlas` is the operator-facing control plane.
- `wiremap` owns reconnaissance, capture, and evidence interpretation.
- `vector` owns ranked action lanes, bounded validation, sessions, and outcomes.
- `intelctl` owns direct shared-intel inspection.
- `labctl` owns build, release, target, and administration workflows.

Do not collapse these domains into one tool unless explicitly instructed.

---

## Current Maturity

Atlas is in an internal engineering/refinement phase.

When the project says a pillar is `ready`, interpret that as:

> ready for internal testing, refinement, and trust hardening.

It does not mean:

- production-ready
- externally audited
- enterprise-ready
- deployment-certified
- autonomous
- cryptographically immutable
- tamper-proof

Use precise language such as:

- ready-to-refine
- internal readiness
- release-trust candidate
- metadata-only
- audit-friendly
- retention-ready
- verifiable

Avoid overclaiming.

---

## Core Philosophy

Atlas favors verifiable integrity over assumed trust.

Every meaningful feature should support at least one of these:

- scope enforcement
- operator control
- evidence integrity
- append-only auditability
- validation discipline
- report clarity
- retention freshness
- release trust
- reproducibility
- known limitations

If a change does not support one of those goals, question whether it belongs.

---

## Safety Boundary

Atlas is for authorized assessment orchestration only.

Do not add features that provide or encourage:

- autonomous exploitation
- persistence
- destructive testing
- credential spraying
- denial-of-service workflows
- stealth/evasion behavior
- out-of-scope target expansion
- malware-like behavior
- unauthorized access

Target-touching workflows must preserve:

1. scope checks
2. capability classification
3. operator intent
4. approval gates when required
5. ledger events
6. evidence handling when applicable

---

## Capability Tiers

Use the existing capability tier model consistently:

- Tier 0: read-only
- Tier 1: passive recon
- Tier 2: active recon
- Tier 3: safe validation, explicit approval required
- Tier 4: intrusive validation, explicit ROE required
- Tier 5: destructive, blocked by default

When unsure, classify higher, not lower.

---

## Development Environment

Use the repository's Nix development environment.

Enter the dev shell with:

```bash
nix-shell
```

The dev shell is expected to provide tools such as:

- `shellcheck`
- `shfmt`
- `bats`
- `git`
- `gpg`
- `jq`
- `fd`
- `rg`
- `rsync`
- `tmux`

Do not assume host-global dependencies are available outside `nix-shell`.

---

## Standard QA Commands

Before declaring a change complete, run the strongest relevant gate.

Preferred full gate:

```bash
nix-shell --run './bin/dev-qa'
```

For targeted checks, use:

```bash
bash -n <changed-shell-file>
git diff --check
nix-shell --run './bin/dev-lint'
nix-shell --run './bin/dev-test tests/atlas.bats'
nix-shell --run './bin/dev-stress'
```

If a command was not run, say so clearly in the final response.

Do not claim tests passed unless they were actually run.

---

## Code Style

Atlas is shell-native.

Follow Bash discipline:

```bash
set -euo pipefail
```

Rules:

- Quote variables.
- Prefer `printf` over `echo` for controlled output.
- Avoid `eval`.
- Avoid parsing `ls`.
- Use `mktemp` for temporary files.
- Clean up temp files with `trap`.
- Check dependencies before using them.
- Keep read-only commands read-only.
- Preserve append-only semantics for audit/event state.
- Use existing helper functions before adding new ones.
- Keep functions small and domain-specific.

Do not introduce large rewrites unless explicitly requested.

---

## File and State Model

Atlas currently favors inspectable file-backed state:

- env records
- NDJSON records
- Markdown packets
- SHA-256 anchors
- metadata-only manifests

Do not migrate to SQLite, server state, hidden caches, or web-backed state unless explicitly requested.

Preferred state direction:

```text
file-backed contracts first
SQLite later only after event, graph, and packet contracts stabilize
```

Important state artifacts include:

- target env records
- operation env records
- scope snapshots
- `ledger.ndjson`
- evidence records
- findings records
- validation records
- reports
- handoff packets
- closeout manifests
- audit packets
- archive packets
- release trust packets

---

## Read-Only Command Rule

Commands documented as read-only must not mutate state.

Read-only examples include:

- `atlas v1 status`
- `atlas op audit`
- `atlas op archive`
- `atlas op verify`
- `atlas op audit-verify`
- `atlas op archive-verify`
- `atlas op trust-chain`
- `atlas release verify`
- `atlas target story`
- `atlas cycle`

If a command writes state, it must be clearly documented as mutating and must write a ledger event when appropriate.

---

## Metadata-Only Packet Rule

These packet types must remain metadata-only unless the project owner explicitly changes the rule:

- handoff packets
- closeout manifests
- audit packets
- archive packets
- release trust packets
- advisor prompt packets
- business-flow evidence packets

Metadata-only packets may include:

- commit hash
- branch
- tag state
- repository cleanliness
- upstream sync state
- v1 readiness JSON
- QA status
- artifact paths
- SHA-256 hashes
- ledger event counts
- freshness states
- known limitations
- retained milestone references
- operation trust-chain replay summaries
- business-flow IDs
- business-flow owner labels
- business-flow system aliases
- business-flow data class labels
- business-flow control objective labels

Metadata-only packets must not include:

- raw runtime artifacts
- target secrets
- session contents
- packet captures
- credential material
- private keys
- tokens
- unredacted evidence bodies
- exploit payloads
- sensitive operator notes

Business-flow evidence is referential evidence. It may point to evidence,
findings, validation, approvals, reports, and packets, but it must not embed
sensitive business data or raw evidence content.

---

## Atlas v1 Readiness

`atlas v1 status` reports internal pillar readiness for testing and refinement.

It must not be treated as production certification.

The v1 pillar contract lives at:

```text
docs/atlas/V1_PILLAR_READINESS.md
```

When touching readiness logic:

- keep status meanings consistent
- preserve `--strict`
- preserve `--json`
- include negative tests
- document required vs optional pillars
- ensure optional planned/disabled pillars do not create misleading failures
- avoid vague readiness claims

A pillar is not ready because Atlas says it is ready. A pillar is ready because Atlas can point to commands, tests, artifacts, reasons, and known limitations.

---

## Production Readiness

`atlas production status` is the stricter production-readiness gate.

It must remain read-only and must not be weakened to make Atlas look finished.

The production contract lives at:

```text
docs/atlas/PRODUCTION_READINESS.md
```

Current production readiness requires, at minimum:

- v1 internal readiness
- clean repository state
- synced upstream state
- current verified release trust packet
- documented production-readiness contract
- signing/provenance evidence
- retained production dry-run or independent validation evidence

If any required production gate is missing, Atlas is `not-ready` for production even if `atlas v1 status` is ready.

---

## Release Trust

The current strategic priority is release trust consolidation.

Release trust work should focus on:

- `atlas release packet`
- `atlas release verify`
- dirty-repo detection
- upstream sync detection
- tag/commit metadata
- v1 readiness JSON
- QA status
- retained milestone references
- known limitations
- metadata-only guardrails
- operation trust-chain binding
- operation trust-chain replay verification
- Markdown/JSON verification parity

Do not embed raw runtime artifacts in release packets.

If adding release verification, include negative tests for:

- dirty repo
- unsynced repo
- missing packet
- stale packet
- commit mismatch
- malformed readiness JSON
- missing QA metadata
- missing operation trust-chain state
- stale operation trust-chain state
- ledger replay mismatch
- archive packet replay mismatch

Do not rely on a release packet's self-reported status when the verifier can replay retained local results.

---

## Testing Expectations

New behavior should include tests.

Preferred coverage:

- positive path
- negative path
- missing artifact path
- stale artifact path
- dirty repository path
- unsynced repository path
- missing dependency path
- strict-mode failure
- JSON validity
- metadata-only packet guardrails
- read-only command non-mutation

A trust gate is only real if it can fail correctly.

---

## Documentation Expectations

Update docs when behavior changes.

Likely files:

- `README.md`
- `tools/atlas/README.md`
- `docs/ATLAS_BLUEPRINT.md`
- `docs/atlas/V1_PILLAR_READINESS.md`
- `docs/atlas/TRUST_LIFECYCLE.md`
- `docs/retention/milestones/MILESTONE_XX.md`
- `docs/retention/releases/`

Documentation must distinguish:

- implemented
- ready-to-refine
- warning
- blocked
- planned
- disabled
- not implemented
- future direction

Avoid overclaiming.

---

## AI Advisor Boundary

Atlas may include an AI Advisor interface, but it is not an execution engine.

Allowed:

- state summaries
- metadata-only prompt packets
- report drafting support
- next-step suggestions
- scope review suggestions
- finding explanation

Not allowed:

- autonomous execution
- direct tool invocation
- scope expansion
- bypassing approvals
- declaring authorization
- reading raw secrets by default
- generating or running exploit logic

External model execution is outside Atlas.

Prefer the phrase:

```text
AI Advisor Interface
```

or:

```text
Advisor Packet Interface
```

when clarity matters.

---

## Atlas OS / ISI / Kernel Boundary

Atlas OS, ISI, and kernel-level work are future layers.

Do not introduce OS-specific or kernel-specific assumptions into the current control plane unless explicitly requested.

Current order:

```text
control plane
trust
retention
release verification
operator clarity
reproducibility
node/runtime later
Atlas OS later
ISI research later
kernel-level work much later
```

Atlas earns deeper layers by proving the current layer.

---

## Web UI Boundary

Do not add a web UI unless explicitly requested.

If a web UI is eventually introduced, it should be:

1. read-only dashboard first
2. approval console second
3. fleet view later
4. execution surface last, if ever

The CLI and file-backed state remain primary.

---

## Command Design

Use predictable command grammar:

```bash
atlas <domain> <verb> [args]
```

Good examples:

```bash
atlas v1 status --strict
atlas op readiness <name>
atlas op audit <name>
atlas op archive <name>
atlas op archive-verify <name> <packet>
atlas op trust-chain <name> --strict
atlas release packet <packet-name>
atlas release verify <packet-name>
atlas target story <target>
atlas evidence add <path>
atlas finding add <title>
atlas validation plan <lane>
```

Avoid vague commands:

```bash
atlas magic
atlas auto
atlas attack
atlas do
atlas runthis
```

---

## Security Language

Prefer:

- verifiable
- auditable
- bounded
- metadata-only
- scope-aware
- approval-gated
- evidence-backed
- ready-to-refine
- local-first
- operator-controlled
- release-trust
- retention-ready

Avoid:

- unhackable
- fully secure
- bulletproof
- autonomous pentest
- exploit automation
- AI hacker
- production certified
- enterprise ready

---

## Public Positioning

If writing public-facing material, describe Atlas as:

```text
Atlas is a shell-native control plane for authorized security assessment workflows.
It helps operators manage scope, evidence, findings, validation, reporting, retention,
and release trust without replacing the underlying domain tools.
```

Do not center offensive capability.

Center:

- authorization
- scope
- evidence
- auditability
- retention
- verification
- operator control

---

## Agent Work Protocol

When working on Atlas:

1. Identify the exact objective or milestone.
2. Inspect the existing command style before editing.
3. Make the smallest coherent change.
4. Preserve architecture boundaries.
5. Add or update tests.
6. Add or update docs.
7. Run the relevant QA gates.
8. Summarize changed files, behavior, tests, and limitations.
9. Do not claim production readiness.

Supporting workflow and validation notes live in:

```text
docs/agents/AGENT_WORKFLOW.md
docs/agents/AGENT_VALIDATION.md
```

---

## Definition of Done

A change is done only when:

- behavior works
- tests cover it
- negative paths are considered
- docs are updated
- QA/lint/tests pass or failures are clearly reported
- limitations are documented
- metadata-only and safety boundaries remain intact
- repo state is summarized clearly

For milestone work, include:

- commit hash
- tests run
- repo cleanliness
- upstream sync state
- tag if applicable
- retention note if applicable

---

## Final Rule

Do not make Atlas more impressive by making it less trustworthy.

Prioritize:

```text
clarity
scope
evidence
verification
retention
release trust
operator control
```

---
> Source: [rodriguezaa22ar-boop/atlas-trust-infrastructure](https://github.com/rodriguezaa22ar-boop/atlas-trust-infrastructure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
