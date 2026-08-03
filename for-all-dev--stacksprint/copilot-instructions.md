## stacksprint

> This file is the source of truth for any Claude Code session working in this

# CLAUDE.md — Verified Inference Stack (VIS) 90-Day Prototyping Sprint

This file is the source of truth for any Claude Code session working in this
repository. Read it fully before making changes. When instructions here
conflict with intuition or upstream examples, this file wins; if it conflicts
with a signed-off spec in `specs/`, the spec wins and this file must be updated.

## 1. Mission

Ship, in 90 days, a working prototype of an ML inference stack whose
**control plane is formally verified** and whose **data plane is formally
isolated**. Concretely: a quantized LLM serving requests end-to-end where the
scheduler, KV-cache allocator, request lifecycle, and memory admission logic
carry machine-checked proofs (Verus), the component architecture runs on seL4
with capability-enforced isolation, and the GPU sits outside the trusted
computing base behind IOMMU-enforced DMA boundaries.

We are NOT verifying: model weights, kernel numerics on GPU, the Python
tooling, or drivers. We ARE verifying: every line of code that decides who
gets memory, whose tokens go where, and what the GPU is allowed to touch.

**The primary deliverable is the SPECS.** Full stop. A complete, expert-signed,
machine-validated specification suite — invariants, transition systems,
refinement structure, and Verus contracts (`requires`/`ensures`/ghost state)
on every control-plane interface — is worth more than any individual green
proof. Proof bodies are deliberately allowed to be admitted (`sorry`-style
`assume`/`admit`): we treat proof completion as a token-spend problem that
future language models and automated proof search will discharge against our
contracts (vericoding / proof-carrying-code posture). Specs are the part that
requires convened human experts and cannot be regenerated later; proofs are
the part that gets cheaper every quarter.

**Definition of success (day 90):** (a) a demo where a small quantized model
serves concurrent streaming requests on the seL4 image under QEMU and on one
reference ARM board; (b) 100% of control-plane interfaces carry signed-off
formal contracts traceable to `specs/invariants.md`, with every spec
*validated* (model-checked, mutation-tested, non-vacuous — see §2.2) even
where its Verus proof is admitted; (c) a proof-debt ledger enumerating every
admitted proof with its contract, ready to hand to automated proof
engineering; (d) the TCB audit document enumerating every trusted line and
every admit. Green proofs are recorded as a burndown metric, not a gate —
except the small "load-bearing set" defined in §2.4.

## 2. Non-negotiable principles

1. **The spec is the deliverable.** Code without its *contract* does not
   merge. Code with a contract but an admitted proof body merges freely —
   that is proof debt, and proof debt is a feature of this sprint, not a
   failure. "Unspecified" is the state that does not exist in this repo.
2. **Admitted ≠ unvalidated.** A `sorry` is only allowed against a spec that
   has been independently validated, because an admitted proof can silently
   hide a vacuous, unsatisfiable, or wrong spec. Validation means all of:
   (i) the corresponding TLA+/Z3 transition system model-checks the same
   invariant (this is cheap and catches most spec bugs); (ii) the spec is
   *satisfiable* — at least one concrete execution trace witnesses it;
   (iii) it kills its assigned mutants at the model level (§8.6). Expert
   sign-off happens on the spec, and only on a validated spec.
3. **Spec before code, spec survives code.** New components start as a TLA+
   or Z3 transition system in `specs/`, get validated per 2.2, get signed
   off, then get a Verus skeleton whose contracts are traceable (by ID: I1,
   I2, …) to the spec. Contracts are written to be *provable by a future
   agent with no context*: self-contained, no reliance on tribal knowledge,
   every ghost abstraction documented at its definition. Assume the entity
   discharging the proof has this repo and nothing else.
4. **A small load-bearing set stays green.** Not everything can be deferred:
   proofs whose failure would invalidate the *architecture* (the kv-alloc
   no-alias/refcount core, the scheduler admission bound, the validator's
   DMA bounds checks) are gate-blocking and must be fully discharged in the
   sprint. The review board owns this list in `specs/invariants.md`; it
   should stay under ~15 invariants. Everything else may be admitted.
5. **Honest debt accounting.** Every admitted proof, every
   `#[verifier::external_body]`, every `assume`, every axiomatized FFI
   boundary gets an entry in `docs/proof-debt.md` or `docs/tcb-ledger.md`
   (debt = we intend to prove it; TCB = we intend to trust it — never
   confuse the two) with an owner, the contract it owes, and a difficulty
   estimate. CI fails if the ledgers and code disagree (`tools/tcb-audit`).
6. **The GPU is an adversary.** Anything returning from the data plane is
   validated by contract-carrying code: shapes, ranges, buffer bounds,
   sequence tags. No control-plane component ever branches on unvalidated
   device data. (Validator contracts are in the load-bearing set.)
7. **Prototype honestly.** Cut scope, not spec rigor. A smaller fully
   specified system beats a larger loosely specified one. When in doubt,
   shrink the demo, never the contracts.

## 3. Architecture

```
                    ┌─────────────────────────────────────────┐
                    │              seL4 kernel                │  proven (existing)
                    └─────────────────────────────────────────┘
   PD = Microkit protection domain          capabilities only, no shared ambient authority
┌──────────┐  ┌───────────┐  ┌───────────┐  ┌────────────┐  ┌─────────────┐
│ net PD   │→ │ gateway PD│→ │ sched PD  │→ │ kv-alloc PD│  │ validator PD│
│ (sDDF,   │  │ (parse,   │  │ (Verus:   │  │ (Verus:    │  │ (Verus:     │
│ unverif.)│  │  Verus)   │  │ batching, │  │ this repo's│  │ DMA result  │
└──────────┘  └───────────┘  │ admission)│  │ module 1)  │  │ checks)     │
                             └───────────┘  └────────────┘  └─────────────┘
                                          │                     ↑
                                          ▼                     │
                             ┌────────────────────────┐   IOMMU-bounded DMA
                             │  GPU / accelerator     │───────────┘
                             │  (untrusted data plane)│
                             └────────────────────────┘
```

Trust levels:
- **Proven:** seL4 kernel and its isolation theorems (upstream proofs).
- **Verified by us:** gateway, scheduler, kv-alloc, validator, request FSM
  (Verus, unbounded proofs) + the inter-PD protocol (TLA+, model-checked).
- **Trusted, unverified, isolated:** sDDF network driver, virtio, boot glue.
  Failures here are contained by capability isolation, not correctness proofs.
- **Untrusted:** GPU stack, model weights, all client input.

## 4. Repository layout

```
vis/
├── CLAUDE.md                  ← this file
├── specs/
│   ├── invariants.md          ← numbered master list (I1…In), owners, status
│   ├── kv_alloc.tla / .py     ← transition systems (Z3 model in proof/ counts)
│   ├── scheduler.tla          ← continuous batching + admission, TLC-checked
│   ├── request_fsm.tla        ← request lifecycle, liveness under fairness
│   └── ipc_protocol.tla       ← inter-PD message protocol
├── crates/
│   ├── kv-alloc/              ← MODULE 1 (shipped, pre-sprint): paged KV
│   │   ├── src/main.rs        │  allocator, ghost page-table map, I1–I4
│   │   ├── proof/inductive.py │  Z3 inductive proof (runs today)
│   │   └── verify.sh
│   ├── scheduler/             ← MODULE 2: continuous batching, admission
│   ├── request-fsm/           ← MODULE 3: lifecycle state machine
│   ├── gateway/               ← MODULE 4: framing/parse, streaming order
│   ├── validator/             ← MODULE 5: data-plane result validation
│   ├── sel4-sys-spec/         ← axiomatized seL4/Microkit bindings (TCB!)
│   └── common/                ← verified shared types, fixed-cap containers
├── platform/
│   ├── microkit/              ← system description, PD manifests, caps
│   ├── qemu/                  ← virt-aarch64 images, run scripts
│   └── board/                 ← reference hardware bring-up
├── dataplane/                 ← UNVERIFIED. CPU int8 backend first; GPU
│   │                            passthrough later. Never imported by crates/.
├── tools/
│   ├── tcb-audit/             ← scans for external_body/assume vs ledger
│   └── seed-bugs/             ← mutation harness for proof red-teaming
├── docs/
│   ├── tcb-ledger.md          ← what we TRUST (axioms, external_body, FFI)
│   ├── proof-debt.md          ← what we OWE (admitted proof bodies + owners
│   │                             + difficulty estimates; vericoding intake)
│   ├── decisions/             ← ADRs, one file per irreversible choice
│   └── demo-script.md
└── ci/                        ← verification-first pipeline (see §8)
```

## 5. 90-day plan

### Phase 0 — Foundations (days 1–10)
- Toolchain image: pinned Verus release, pinned Rust toolchain, TLC, Z3,
  Microkit SDK, aarch64 cross toolchain. One `Dockerfile`, hash-pinned.
  Everything below runs in this image; no "works on my machine."
- `specs/invariants.md` v1 signed off, including the load-bearing set (§2.4).
- Module 1 (kv-alloc): contracts finalized; its Z3 model already validates
  the spec (non-vacuity + mutant kill demonstrated pre-sprint). Its Verus
  proofs are in the load-bearing set — iterate to green here, both because
  the architecture rests on them and as calibration data for how much proof
  effort each invariant class costs (feeds the proof-debt estimates).
- CI skeleton: *contracts* are the build — a crate whose signatures don't
  carry spec-traceable contracts doesn't compile; admitted bodies pass with
  a ledger entry.
- **Gate G0:** kv-alloc green; invariants v1 signed; Microkit hello-world
  boots in QEMU.

### Phase 1 — Verified control plane, hosted (days 11–35)
- Modules 2–4 spec'd in TLA+ (TLC-checked incl. liveness under fairness:
  no starvation, every request reaches a terminal state), then implemented
  in Verus. Run hosted on Linux first — same crates, mock IPC.
- Scheduler invariants (owner: verification lead): admission never exceeds
  proven memory bound; preemption preserves sequence state; work conservation.
- End-to-end hosted demo: gateway → scheduler → kv-alloc → CPU int8
  data plane serving a small model, streaming, concurrent.
- **Gate G1:** hosted demo + all Module 1–4 *contracts* signed and validated
  (TLC clean, satisfiability witnesses, model-level mutants killed) +
  load-bearing proofs green. Non-load-bearing proof bodies may be admitted
  with ledger entries.

### Phase 2 — seL4 integration (days 36–60)
- `sel4-sys-spec`: axiomatize the Microkit API surface we use (channels,
  memory regions, notifications). Every axiom → tcb-ledger. Expert review
  by the seL4 consultant is the acceptance test.
- Port crates into PDs; capability manifest review (least authority per PD).
- sDDF network driver integration (unverified, isolated); virtio for QEMU.
- Inter-PD protocol model-checked (`ipc_protocol.tla`) against deadlock and
  message loss; Verus endpoints prove they refine the protocol's local views.
- **Gate G2:** end-to-end serving inside QEMU/seL4, CPU data plane.

### Phase 3 — Data plane isolation + hardening (days 61–82)
- Validator PD (Module 5): verified bounds/shape/tag checks on everything
  crossing the DMA boundary.
- IOMMU/SMMU configuration for device passthrough on the reference board;
  GPU (or NPU) behind it if board support allows — otherwise ship CPU/NPU
  data plane and document the GPU path as designed-not-integrated. Decide by
  day 65, ADR required.
- Red-team week — this is the *spec* audit and the sprint's most important
  quality event: `tools/seed-bugs` mutates the models and the code
  (off-by-one, missing refcount, double-free, admission overshoot). Kill
  criteria differ by proof status: mutants in green-proof code must be
  killed by a failing proof; mutants in admitted-proof code must be killed
  at the model level (TLC/Z3) — that's the check that the spec itself would
  have caught the bug once proofs are discharged. A surviving mutant is a
  spec gap: fix the spec, re-validate, re-sign. Spec gaps found here are
  gold; log every one in `docs/decisions/`.
- Performance sanity pass (§9): we are not chasing throughput, but the demo
  must not be embarrassing. Batch size, tokens/sec, p99 recorded.
- **Gate G3:** red-team report clean; board demo runs.

### Phase 4 — Spec package, audit, demo (days 83–90)
- **The spec package** — the flagship artifact: the full validated spec
  suite + contract-carrying codebase + proof-debt ledger, packaged so that
  an automated proof-engineering campaign (or a future model, or a proof
  contractor) can discharge the debt with zero tribal knowledge. Includes
  per-invariant difficulty estimates calibrated from the load-bearing set,
  and a harness that accepts a candidate proof and reports which ledger
  entries it retires (proof-carrying-code intake path).
- TCB audit doc: trusted lines vs. proof debt, clearly separated; diagram
  of contract coverage (target: 100%) vs. proof coverage (whatever it is —
  report it honestly).
- Demo script rehearsed: live seeded-bug → failing model check / failing
  proof is part of the demo; also demo the intake harness retiring one
  admitted proof with a machine-generated proof body if feasible.
- Technical report draft suitable for external experts to critique.

Schedule risk policy: gates slip features, never rigor. If G2 is late, Phase 3
drops GPU passthrough first, then the board target, never the proofs.

## 6. Expert convening (assume budget; recruit early, days 1–15)

- **seL4/Microkit consultant** (ideally ex-Trustworthy Systems): reviews
  capability manifests, sel4-sys-spec axioms, sDDF integration. ~2 days/week.
- **Verus expert** (Secure Foundations orbit): unblocks proof engineering,
  reviews ghost-state architecture, triggers, trait encodings. Embedded
  weeks 2–8, on call after.
- **TLA+ specialist**: owns liveness specs and fairness assumptions;
  reviews refinement claims between TLA+ specs and Verus invariants.
- **Inference systems engineer** (vLLM/TensorRT-LLM background): keeps the
  scheduler/allocator design honest against real serving behavior
  (preemption, prefix caching, chunked prefill).
- **IOMMU/platform security reviewer**: audits SMMU config and the DMA
  boundary assumptions in Phase 3.
- Standing review board: 90-minute proof-and-spec review twice weekly.
  No spec is "signed off" (§2.3) without a board entry in `docs/decisions/`.

## 7. Working rules for Claude Code sessions

- **Always run verification.** `./ci/verify-all.sh` locally before proposing
  a merge (admits pass; contract or load-bearing failures don't). Single
  crate: `cd crates/<c> && ./verify.sh`. Specs: `./ci/tlc.sh specs/<spec>.tla`.
- **Never weaken a spec to make a proof pass — admit instead.** If an
  invariant resists proving, leave the contract intact, admit the body,
  file the ledger entry, move on. If you believe the *spec* is wrong,
  stop and write up why in the PR; only the review board changes signed
  specs. Spec changes invalidate sign-off and require re-validation (§2.2).
- **Write contracts for a reader with no context.** Every `requires`/
  `ensures`/ghost definition must be understandable and provable from the
  repo alone — future proof agents get no Slack history. If a contract
  needs an explanation, the explanation goes in a doc comment at the
  definition, now.
- **No new admit/`assume`/`external_body` without a same-commit ledger
  entry** (proof-debt.md for admits, tcb-ledger.md for trusted code). CI
  enforces this; never bypass the audit tool. Never admit anything in the
  load-bearing set.
- **Traceability comments are mandatory:** every proven `ensures` clause that
  discharges a master invariant carries `// [I<n>]`.
- **Verus style:** small `spec fn`s; name ghost state after the abstraction it
  mirrors (`tables`, not `g1`); prefer `=~=` extensional-equality proofs over
  quantifier gymnastics; broadcast groups over ad-hoc lemma imports; keep
  `proof {}` blocks adjacent to the mutation they justify.
- **Unverified code is quarantined** in `dataplane/` and `platform/` driver
  dirs. `crates/` must not depend on it; the dependency direction is checked
  in CI.
- **Panics are proof failures.** Verified crates prove absence of panics;
  do not add `unwrap`/`expect`/indexing without bounds proofs.
- **Model updates:** if you change a transition in a Verus crate, update the
  corresponding TLA+/Z3 model in `specs/` in the same PR, and vice versa.
  Divergent models are worse than no models.
- Commit style: `module: what changed [invariants touched] (proof status)`.
- When toolchain downloads fail in sandboxed environments, the needed egress
  domains are: `release-assets.githubusercontent.com` (Verus releases),
  `static.rust-lang.org` (pinned toolchain), plus the standard crates.io set.

## 8. CI (verification-first)

Pipeline order — earlier stages are cheaper and gate later ones:
1. `tcb-audit` (seconds) — both ledgers vs. code; load-bearing set has
   zero admits.
2. Verus verify, all crates, parallel (minutes) — contracts must check and
   load-bearing proofs must discharge; admitted bodies pass. This IS the
   type check.
3. Spec validation on changed specs: TLC + satisfiability witnesses +
   vacuity checks. Full TLC nightly (liveness is slow).
4. Hosted integration tests (mock IPC) — with proofs admitted, runtime
   testing is doing real work again; contracts double as runtime assertions
   in test builds (`--check-contracts`), so admitted invariants are at
   least dynamically monitored until proven.
5. QEMU/seL4 boot + smoke serve (nightly + pre-merge on platform changes).
6. Nightly mutation run (`seed-bugs`) per §5 Phase 3 kill criteria; new
   surviving mutants page the verification lead.
7. Nightly proof-debt burndown report: admits by module, difficulty
   histogram, trend. Debt may grow during the sprint; *untracked* debt
   may not exist for a single commit.

## 9. Explicit non-goals (rejecting these in review saves weeks)

- Competitive throughput/latency vs vLLM. Record numbers; do not optimize
  past "demo is smooth."
- Verified GPU kernels, verified numerics, or floating-point soundness
  proofs. Data plane is int8 CPU first; correctness there is *checked at the
  boundary*, not proven inside.
- Multi-node serving, tensor parallelism, speculative decoding.
- Verifying the sDDF drivers or virtio. Isolation is their containment story.
- Confidentiality/side-channel claims. seL4's infoflow proofs exist but our
  configuration will not inherit them in 90 days; do not imply otherwise in
  any writeup.

## 10. Module 1 status (pre-sprint, already in repo)

`crates/kv-alloc`: paged KV-cache block allocator with refcounted prefix
sharing and CoW. Ghost `Map<SeqId, Set<BlockId>>` mirrors logical page
tables. Invariants I1 (free ⟺ rc=0), I2 (rc exactness), I3 (free-list
no-dup), I4 (range). Z3 inductive proof of the transition system passes
today, including refutation of a seeded fork-without-refcount bug with a
concrete use-after-free trace — i.e., this module's spec is already
validated per §2.2. Its invariants are in the load-bearing set (§2.4): the
Verus file is written but not yet checker-iterated (network-restricted
authoring environment); Phase 0 owns getting it green and using the effort
as the first calibration point for proof-debt difficulty estimates.

---
> Source: [for-all-dev/stacksprint](https://github.com/for-all-dev/stacksprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
