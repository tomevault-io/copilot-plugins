## peat

> Top-level skill for Claude Code sessions across any peat-* repo. Read first, then read the per-repo SKILL.md.


# Peat Ecosystem SKILL

Peat is an interoperability-first mesh registry sync platform built for heterogeneous autonomous systems in defense and edge environments. Its core value proposition is **interoperability that enables scale** — Peat connects systems that don't speak the same language across transport and protocol boundaries. Peat is developed under the Defense Unicorns GitHub org: https://github.com/defenseunicorns

## When this skill applies

- Any session touching files in a `peat-*` repo or the top-level `peat` crate
- Cross-repo changes affecting more than one peat repo
- Reviewing a PR in any peat repo

After reading this file, read the relevant per-repo SKILL.md from the router below. Per-repo skills are authored against `peat/SKILL_TEMPLATE.md`.

## Skill router

Read only what's relevant to the current task. Do not preload every per-repo skill.

The ecosystem comprises **one Rust workspace repo** (`peat`) plus several sibling repos. Internal workspace subcrates do **not** have separate per-repo skills — they share this file.

**Sibling repos** (each its own git repo, each with its own per-repo skill):

| Repo | Purpose | Skill |
|---|---|---|
| **peat** | Rust workspace. Hosts the ecosystem skill (this file) and the per-repo skill for the workspace. Active. | This file |
| **peat-mesh** | Mesh networking; pluggable transport, peer discovery, topology, routing, optional Automerge/Iroh sync, optional HTTP/WS broker. **Top-tier active.** | `peat-mesh/SKILL.md` |
| **peat-btle** | BLE transport bridge. M5Stack/ESP32 integration. | `peat-btle/SKILL.md` |
| **peat-lite** | Lightweight implementation for constrained environments. | `peat-lite/SKILL.md` |
| **peat-atak-plugin** | One consumer-plugin integration (Android/Kotlin). Pure Kotlin/Gradle — no Rust here. Consumes pre-built JNI/UniFFI bindings from `peat/peat-ffi`. Repo name is historical; this repo treats it as a generic consumer surface. | `peat-atak-plugin/SKILL.md` |
| **peat-sim** | ContainerLab-based network simulation harness. Not on the production path. | `peat-sim/SKILL.md` |

**Workspace subcrates** (members of the `peat` repo, share this skill):

| Subcrate | Role |
|---|---|
| `peat` | Top-level crate; eventual home for shared types and traits. Currently a placeholder (see "`peat` repo-specific skill" below). |
| `peat-schema` | Schema definitions. |
| `peat-protocol` | Protocol logic (`Translator`, `ChangeEvent`, etc.). |
| `peat-transport` | Transport abstractions used by the workspace. |
| `peat-persistence` | Persistence layer. |
| `peat-ffi` | FFI bindings for Kotlin/Swift (UniFFI 0.28, proc-macro mode) + direct `jni 0.21` for Android. The only routinely-unsafe Rust in the ecosystem. |

**Status unknown — confirm with Kit before authoring skills.** The following were listed in earlier drafts but aren't currently visible from this checkout: `peat-registry`, `peat-node`, `peat-gateway`, `peat-rmw`, `peat-mavlink`. They may be planned, renamed, deprecated, or in private repos. Per the active-repos record, only `peat` and `peat-mesh` are currently top-tier active.

## Hard invariants (cross-cutting)

These rules apply in every repo. Violating one without explicit user approval is out of scope, full stop.

**Language.** Rust everywhere except Kotlin in consumer plugins (currently `peat-atak-plugin`). No new language dependencies. No Python. No shell scripts for anything that belongs in Rust.

**FIPS-approved cryptographic primitives only.** Every algorithm used anywhere in the peat ecosystem must be on the FIPS 140-3 approved list. AEAD: AES-GCM (not ChaCha20-Poly1305). Signatures: Ed25519 (FIPS 186-5) or ECDSA-P256/P384. Key agreement: ECDH-P256/P384 (X25519 only with explicit review). KDF: HKDF-SHA-2. MAC: HMAC-SHA-2. TLS/QUIC must run under a FIPS-mode crypto provider (e.g. `aws-lc-rs` for rustls; the default `ring` is **not** FIPS-validated). MLS suites must be FIPS-aligned (e.g. `MLS_128_DHKEMP256_AES128GCM_SHA256_P256`). Existing ChaCha20-Poly1305 references in ADR-006/044/048/049 + spec docs are tracked for amendment; do not propagate them. Canonical reference: ADR-060 §5 "Cryptographic primitives (FIPS posture)" + `CLAUDE.md` § "Hard rule: FIPS-approved cryptographic primitives only".

**No consumer-specific references in peat.** peat is the generic mesh substrate; consumers (mobile-app plugins, wearable firmware, CLI tools, server bridges) live in sibling repos. Code, comments, examples, READMEs, operational docs, JNI symbol names, package paths, and test fixtures in this repo MUST NOT name a specific consumer (ATAK, WinTAK, iTAK, WearTAK, etc.) or vendor-derived identifiers. Use "consumer", "consumer plugin", "CoT consumer", "mobile-app plugin", "wearable", "CLI tool", or "server bridge". Protocol-name appearances (e.g. CoT, TAK Server wire protocol) are allowed where the protocol itself is structurally load-bearing; consumer-name appearances are not. The only exception is ADRs in `docs/adr/` citing real-world use cases that motivated a design decision. See `CLAUDE.md` § "Hard rule: no consumer-specific references in peat" for the full rule + rationale. Verification gate (below) includes a grep check for ATAK / vendor names in new diffs.

**Dependency flow.** `peat` is the dependency anchor. Common types, traits, error handling flow *down* from `peat`. Repos depend on `peat`, never on each other directly. Circular dependencies are rejected.

**Transport agnosticism.** Peat protocol logic must not assume a transport. BLE, mesh, IP, serial are all interchangeable. Transport-specific code stays in transport repos (`peat-btle`, `peat-mesh`) or the `peat-transport` workspace subcrate — never in core / protocol / persistence layers or non-transport sibling repos.

**Interoperability first.** Every feature decision answers: does this make Peat more or less interoperable with external systems? Peat must never require a counterpart to run Peat software to integrate.

**Unsafe Rust.** Requires explicit justification in a code comment. The FFI boundary in `peat-ffi` (workspace subcrate inside the `peat` repo) is the only routinely legitimate unsafe zone.

**FFI boundary direction.** All Peat protocol logic stays in Rust (`peat-ffi` and the rest of the workspace). Kotlin in `peat-atak-plugin` is UI and Android lifecycle only — never a destination for protocol, mesh, transport, persistence, or serialization logic. Native libraries and Kotlin bindings flow from `peat-ffi` into `peat-atak-plugin`, not the reverse.

**Async runtime.** Tokio is the ecosystem standard. The `peat` workspace pins `tokio = { version = "1", features = ["full"] }` (or feature subsets where appropriate); sibling repos do the same. Do not introduce alternative async runtimes (`async-std`, `smol`) without explicit user approval.

**Error handling.** Library crates use `thiserror` for crate-level error enums. Application/binary code uses `anyhow` for ergonomic error chaining. Both are pinned at `1` in workspace deps and in sibling repos — use those, do not pull alternative error crates without justification.

**Commit signing.** GPG-signed commits are required across the ecosystem. Do not configure git to bypass signing or use `--no-verify` to skip pre-commit hooks. Fix the signing setup; do not work around it.

**Branch + merge convention.** Trunk-based development on `main` with short-lived feature branches. Every PR squash-merges to `main` — write the PR title as if it will become the merge commit subject (because it does).

## Workflow

For any task in a peat repo:

1. **Orient.** Read this file. Read the per-repo SKILL.md from the router. Read `CLAUDE.md` if present. Run `git status` and `git log -10`.
2. **Locate the spec.** Confirm the task has a GitHub issue with the required sections (see "Issue format" below). If not, ask the user before implementing.
3. **Plan.** Produce a short plan. Check it against the hard invariants and the per-repo skill's scope guards.
4. **Implement** following the per-repo workflow. Vertical slices, one concern per commit.
5. **Verify** per the per-repo skill's exit criteria.
6. **Hand off.** Open PR referencing the issue. Summary states *what changed and why*. Flag any cross-repo implications.

## Verification (ecosystem-level)

Beyond the per-repo verify checklist, an ecosystem-level change is not done until:

- [ ] Each affected repo's CI is green
- [ ] No new cross-repo cycle introduced (`peat` does not depend on its consumers; sibling repos do not depend on each other)
- [ ] PR references a GitHub issue with Context / Scope / Acceptance Criteria / Constraints / Dependencies sections
- [ ] If a hard invariant was waived, the PR description names which one and quotes the user approval
- [ ] **For changes inside the `peat` repo:** the new diff is free of consumer-specific identifiers (ATAK, WinTAK, iTAK, WearTAK, etc.). Run `git diff main -- ':!docs/adr' ':!docs/whitepaper' ':!CLAUDE.md' ':!SKILL.md' ':!CHANGELOG.md' | grep -E '^\+' | grep -iE '\b(atak|wintak|itak|weartak)\b' | grep -vE 'peat-atak-plugin|com\.atakmap|atakmap\.app|ATAKActivity'` — must be empty before merge. The pipeline checks ADDITIONS only (`^\+`); excludes the rule documents themselves (which legitimately enumerate the forbidden names as part of the rule definition), the release-notes archive (`CHANGELOG.md`, which by purpose records the history of vendor-name removals and may need to name what was removed to be useful), and ADR / whitepaper material; and tail-filters the sibling repo name `peat-atak-plugin` (its actual repo name, historical) plus the third-party host app's real Android identifiers (`com.atakmap.*`, `ATAKActivity`) that operational adb commands genuinely target. If a citation is genuinely load-bearing it belongs in an ADR (`docs/adr/`) or a CHANGELOG entry, not in code or operational docs.

"Seems right" or "the diff looks correct" is never sufficient.

## Anti-rationalization

| Excuse | Rebuttal |
|---|---|
| "This change spans repos but they're tightly coupled — one big PR is cleaner." | One PR per repo, linked through a tracking issue. Atomicity across repos isn't real; reviewability is. |
| "I'll just import directly from `peat-mesh` into `peat-node` for this." | Cross-repo direct deps are a circular-dependency factory. Add the trait to `peat` core. |
| "It's only a tiny shell script for a build helper." | No shell scripts for things that belong in Rust. Justify the language choice with the user first. |
| "I'll write a quick TS/Python utility for this." | No new language dependencies without explicit approval. |
| "`peat` core doesn't have the type I need; I'll just put it in this repo for now." | Surface the gap as an issue against `peat`. Don't fork the type system. |
| "I'll move this Peat protocol logic into Kotlin to make the FFI simpler." | All Peat protocol logic stays in Rust. Kotlin is UI and Android lifecycle only. |
| "This change makes Peat assume the counterpart is also running Peat — fine for now, we'll generalize later." | Interoperability-first is the product. Generalize before merging, or don't merge. |
| "Just one mention of ATAK in the comment is fine — that's the consumer everyone knows about." | No consumer-specific references in peat. Ever. Use "consumer plugin" / "CoT consumer" / "mobile-app plugin". One mention becomes 382 mentions in two years. Verification gate greps for it. |
| "The JNI symbol name has to encode the Java package path of the calling class." | True for the suffix; the Java package path itself is a choice. Pick a generic namespace (`com.defenseunicorns.peat.PeatJni`), not a consumer-specific one (`com.defenseunicorns.<vendor>.peat.PeatJni`). |

## Scope guards

- Kit is the general contractor across all repos. Claude Code sessions are sub-contractors, scoped to **one repo at a time**.
- Cross-repo changes are coordinated through linked GitHub issues, not by reaching across repos.
- Use the public API/traits of other peat repos. Never assume their internals.
- Do not add a dependency on `peat` core that assumes types or traits not yet stabilized — flag assumptions in the PR.
- Do not add new repos to the ecosystem without explicit user approval.

## Issue format

Each issue used as a Claude Code spec must include:

```
## Context
Which repo(s) this touches and why.

## Scope
What is in scope. What is explicitly out of scope.

## Acceptance Criteria
Specific, testable conditions for done.

## Constraints
Architecture invariants, performance requirements, conventions.

## Dependencies
Links to related issues or PRs in other repos.
```

## Gotchas

Populate as sessions run. One line per gotcha plus a `Why:` line.

- Cross-repo changes: develop with `[patch.crates-io]` path overrides pointing at local sibling checkouts; run the release chain only after the change works end-to-end against the consumer.
  Why: serializing through publish-per-layer (PR → merge → tag → crates.io → bump consumer) costs hours per round-trip and you can't iterate the consumer until the upstream publish lands; the override workflow caught a `tokio::spawn` concurrency race and a cross-version legacy-read BLOCKER before either reached a release (peat#864 chain).
- A single surface symptom can have multiple independent root causes at different layers — keep bisecting after the first fix lands instead of declaring victory.
  Why: peat#864 ("`subscribe_progress` emits no terminal frame") was three separate defects — a peat-mesh sync_cooldown silent-drop, a missing peat-protocol watcher, and a wholesale-scalar Automerge merge race — each fixed in a different repo; closing the issue after the first fix would have been wrong.
- Don't close an issue when the wire-up lands but the end-to-end acceptance test still fails; reopen with a narrowed title rather than filing a vague follow-up.
  Why: peat#864 was prematurely closed at the rc.7 wire-up while its named acceptance test still stalled; reopening with the narrowed scope kept the bisect trail intact.

---

# `peat` repo-specific skill

This repo is a **Rust workspace** with seven internal subcrates plus example crates. It also hosts the ecosystem skill above.

## Workspace members

Subcrates inherit the workspace `Cargo.toml`'s pinned deps (Tokio, serde, thiserror, anyhow, etc.). Cross-subcrate changes coordinate **within** this repo; cross-repo changes follow the ecosystem skill's "one PR per repo" rule.

- `peat` — top-level crate; eventual home for shared types and traits. Currently a placeholder (see below).
- `peat-schema` — schema definitions.
- `peat-protocol` — protocol logic (`Translator`, `ChangeEvent`, etc.).
- `peat-transport` — transport abstractions used by the workspace.
- `peat-persistence` — persistence layer.
- `peat-ffi` — FFI bindings (see "FFI conventions" below).
- `examples/peat-tak-bridge`, `examples/peat-ble-test` — workspace examples.
- `examples/m5stack-core2-peat` is **excluded** from the workspace (separate toolchain — embedded ESP32 / xtensa-esp-none-elf).

## `peat` top-level crate (placeholder)

The `peat` subcrate is currently a placeholder. Its intended role is to be the shared dependency that gives the ecosystem a single source of truth for types, traits, and errors.

**Intended contents:**
- Core data types (messages, identities, capabilities)
- Shared traits (e.g., `Transport`, `Node`)
- Error types

**Does NOT belong in `peat` core:**
- Transport implementations (live in `peat-transport`, `peat-mesh`, `peat-btle`)
- Hardware-specific code
- Platform-specific code (Android, ROS2)

Until `peat` core stabilizes, sessions should be conservative about adding dependencies on it and must flag any assumptions about peat-core types in PR descriptions.

## `peat-ffi` subcrate — FFI conventions

The FFI boundary uses two complementary tools:

- **UniFFI 0.28 in proc-macro-only mode** (no UDL file) — generates Kotlin and Swift bindings from Rust types annotated via `#[uniffi::export]` etc. Async-supported via UniFFI's `tokio` feature.
- **Direct `jni 0.21` bindings** for Android native methods that bypass JNA's symbol-lookup issues. These coexist with UniFFI; use UniFFI as the default and `jni` only where JNA fails.

JNA (`net.java.dev.jna:jna:5.14.0@aar`) is the runtime layer UniFFI depends on at the Kotlin side and is required in `peat-atak-plugin/app/build.gradle.kts` — do not remove it.

`peat-atak-plugin` consumes pre-built native libraries (copied into `app/libs`) and copied Kotlin bindings (into `app/src/main/java/uniffi/peat_ffi`) from this crate's build output. **Do not duplicate FFI types or bindings inside `peat-atak-plugin`** — `peat-ffi` is the single source of truth.

## Workflow guards for changes to `peat` core or `peat-ffi`

- Any new public type or trait in `peat` core requires a brief design note in the PR description.
- Any breaking change to a public item in `peat` core or to `peat-ffi`'s FFI surface requires a list of downstream consumers that need updating, in the PR description.
- Removing a public item requires confirming via `cargo check -p <consumer>` (and, for FFI, rebuilding `peat-atak-plugin`) that no consumer breaks, OR coordinating an update PR in each consumer first.

---

# Open questions (for Kit)

Resolved in this PR: error handling (`thiserror` + `anyhow`), async runtime (Tokio), branch + merge convention (trunk-based + squash-merge), commit signing (GPG required), FFI tooling (UniFFI 0.28 + `jni 0.21`).

Still open:

- One-sentence statement on Peat's relationship to UDS (peer / complement vs. subset).
- Confirm status (active / planned / deprecated / renamed) for repos flagged **status unknown** in the router: `peat-registry`, `peat-node`, `peat-gateway`, `peat-rmw`, `peat-mavlink`. If they exist privately, the router should note that; if they were renamed, the router should reflect the current name; if deprecated, drop them.
- WearTAK integration location — inside `peat-atak-plugin` or a separate repo?
- FFI threading model — once stabilized, document the Kotlin-coroutine ↔ Rust-async convention via UniFFI's `tokio` feature in the `peat-ffi` section above.
- Required reviewers per repo / CODEOWNERS — none currently in `peat` repo. Adopt CODEOWNERS, or document the review-routing convention in the ecosystem skill.

---
*Last updated: 2026-05-05*
*Maintained by: Kit Plummer, VP Data and Autonomy, Defense Unicorns*

---
> Source: [defenseunicorns/peat](https://github.com/defenseunicorns/peat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
