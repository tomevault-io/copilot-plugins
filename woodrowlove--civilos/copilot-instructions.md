## civilos

> AxiaSystem path:      ../AxiaSystem

# CivilOS — Claude Code Project State

## Ecosystem
AxiaSystem path:      ../AxiaSystem
Rust Bridge path:     ../AxiaSystem-Rust-Bridge
Target system path:   .
Registry path:        ../axia-ecosystem/docs/AXIA_API_REGISTRY.md

## Harness documents
Primary harness:      ../axia-ecosystem/docs/harness/CIVIL_SYSTEM_BUILD_HARNESS_v2.1.md
Domain model:         ../axia-ecosystem/docs/harness/LAYER_2_DOMAIN_KNOWLEDGE_COMPRESSION.md
Capability taxonomy:  ../axia-ecosystem/docs/harness/LAYER_3_CAPABILITY_TAXONOMY.md
Failure playbook:     ../axia-ecosystem/docs/harness/LAYER_4_FAILURE_RECOVERY_PLAYBOOK.md
Decision policy:      ../axia-ecosystem/docs/harness/LAYER_5_AUTONOMOUS_DECISION_POLICY.md
Session continuity:   ../axia-ecosystem/docs/harness/LAYER_6_SESSION_CONTINUITY_PROTOCOL.md

## Session documents
Session state:        SESSION_STATE.md
Session index:        docs/sessions/SESSION_INDEX.md
Handoff location:     docs/sessions/

## CivilOS design documents (Phase 0 + Phase 2)
System intent:        docs/civilos-design/SYSTEM_INTENT.md
Actor map:            docs/civilos-design/ACTOR_MAP.md
Capability surface:   docs/civilos-design/CAPABILITY_SURFACE.md
Workflow map:         docs/civilos-design/WORKFLOW_MAP.md
Capability contracts: docs/civilos-design/CAPABILITY_CONTRACTS.md (Batch 1: 5 contracts)
Capability contracts: docs/civilos-design/CAPABILITY_CONTRACTS_BATCH2.md (Batches 2-7: 83 contracts)
Resolution map:       docs/civilos-design/CAPABILITY_RESOLUTION_MAP.md (Batch 1: 5 resolutions)
Resolution map:       docs/civilos-design/CAPABILITY_RESOLUTION_MAP_BATCH2.md (Batches 2-7: 83 resolutions)

## Truth control documents
Capability status:    docs/implementation/CAPABILITY_STATUS.md
Phase summary:        docs/implementation/PHASE_SUMMARY.md
Known limitations:    docs/implementation/KNOWN_LIMITATIONS.md
Open questions:       docs/implementation/OPEN_QUESTIONS.md
Failure log:          docs/implementation/FAILURE_LOG.md
Decision log:         docs/implementation/DECISION_LOG.md
Implementation notes: docs/implementation/IMPLEMENTATION_NOTES.md

## Build commands

AxiaSystem:
  build:   cd ../AxiaSystem && dfx build
  test:    cd ../AxiaSystem && bash scripts/golden_path_resolve_subject.sh
  deploy:  cd ../AxiaSystem && dfx deploy wallet && dfx deploy admin2 && dfx deploy namora_ai && dfx deploy governance && dfx deploy escrow && dfx deploy nft && dfx deploy payout

Rust Bridge:
  build:   cd ../AxiaSystem-Rust-Bridge && cargo build
  test:    cd ../AxiaSystem-Rust-Bridge && cargo test
  run:     cd ../AxiaSystem-Rust-Bridge && cargo run

Target system (CivilOS):
  build:   N/A — CivilOS is documentation + orchestration at this stage
  test:    N/A — tests run against AxiaSystem canisters via dfx
  run:     N/A

## Canister IDs (local replica — regenerated on each dfx start --clean)

Last deployed: 2026-03-27

user:       uzt4z-lp777-77774-qaabq-cai
identity:   uxrrr-q7777-77774-qaaaq-cai
wallet:     umunu-kh777-77774-qaaca-cai
token:      u6s2n-gx777-77774-qaaba-cai
admin2:     ucwa4-rx777-77774-qaada-cai
namora_ai:  vg3po-ix777-77774-qaafa-cai
governance: vpyes-67777-77774-qaaeq-cai
escrow:     vizcg-th777-77774-qaaea-cai
nft:        vt46d-j7777-77774-qaagq-cai
payout:     v27v7-7x777-77774-qaaha-cai

Note: IDs change on every dfx start --clean. After redeployment, run the full
configuration sequence documented in docs/implementation/IMPLEMENTATION_NOTES.md.

## Build state
Current phase:         CivilOS Phase 4 — Domain Layer + Full Contract Coverage
Last completed gate:   88 contracts, 83 classified, 5 domain registries, 10 workflows, golden path proven
Current capability:    56 ORCHESTRATION_READY, 26 DOMAIN_EXTENSION_NEEDED (registries built), 1 SYSTEM_PRIMITIVE
Current state:         18/18 tests pass, full end-to-end proven

## Capability summary

| # | Capability | State |
|---|-----------|-------|
| 1 | resolve_subject | Proven |
| 2 | authenticate_subject | Proven |
| 3 | resolve_system_state | Proven |
| 4 | evaluate_legitimacy | Proven |
| 5 | attest_action | Proven |
| 6 | explain_decision | Proven |
| 7 | execute_transfer | Live |
| 8 | resolve_financial_state | Not Started |
| 9 | register_asset | Not Started |
| 10 | establish_governance_context | Not Started |
| 11 | record_governance_decision | Not Started |

## Open questions
NONE

## Known blockers
NONE

## Last handoff
Session: 20260328-006
Summary: Batch 1 build order complete. 6 bridge exposures in AxiaSystem-Rust-Bridge (evaluate_legitimacy, execute_transfer, register_asset, record_governance_decision, attest_action, explain_decision). CivilOS Rust crate created with SpineClient + issue_driver_license workflow (4-step composed chain). All code compiles, all unit tests pass.
Next action: Live end-to-end verification (requires dfx start), then Batch 2 workflows or capability contracts.

## Notes
All 11 proven capabilities live in the AxiaSystem User canister (user/main.mo).
6 capabilities now bridge-exposed in AxiaSystem-Rust-Bridge (9 FFI exports total).
CivilOS repo contains Rust orchestration crate (src/), documentation, truth ledger, and agent governance.
CivilOS depends on axia_system_rust_bridge crate via path dependency.

---
> Source: [WoodrowLove/civilOS](https://github.com/WoodrowLove/civilOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
