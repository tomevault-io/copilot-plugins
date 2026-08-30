## ix-rust

> Build IX v2 as a simple, clean, capability-complete Rust search engine and benchmark platform. Simplicity is for tidiness and slop avoidance, never for capability loss. Primary performance objective: become faster than ripgrep and other top search competitors (for example fff and codedb) on transparent benchmark suites, with a tracked target of at least 50% faster versus ripgrep on agreed workloads.

# AGENTS.md

## Mission
Build IX v2 as a simple, clean, capability-complete Rust search engine and benchmark platform. Simplicity is for tidiness and slop avoidance, never for capability loss. Primary performance objective: become faster than ripgrep and other top search competitors (for example fff and codedb) on transparent benchmark suites, with a tracked target of at least 50% faster versus ripgrep on agreed workloads.

## Core Patterns (Required)
- `ATOMIC`: ship one coherent slice at a time.
- `DRY`: no duplicated ownership logic.
- `STEP`: deterministic sequence, no random side quests.
- `SOLID`: stable contracts and clear module boundaries.
- `LEVER`: maximize impact with minimum durable architecture change.
- `YAGNI`: no speculative systems without present value.
- `MODULAR`: narrow ownership per module.
- `REUSE`: prefer proven patterns and reusable components.
- `UNIFIRM`: consistent naming, contracts, and behavior.
- `PARITY`: CLI, stats, tests, and docs stay aligned.

## Anti-Patterns (Forbidden)
- Workarounds.
- Assumption-driven changes without direct evidence.
- File sprawl.
- Parallel systems for the same responsibility.
- Prompt scaffolding leakage into user-visible output.
- Non-problem solving.
- Generic or vague implementation notes.
- Fallback behavior that hides unsupported capability.

## Operating Principles
- Observe > Reflect > Make.
- Keep focus on the active objective.
- Maintain strict directory depth hygiene with self-explanatory naming.
- Favor compile-time boundaries over runtime shims.
- Minimum change means minimum durable architecture change, not minimum typing.
- When a performance regression resists local explanations, zoom out from the immediate branch or gate and re-price the whole workload shape before editing more control flow. Check retained bytes, slowest files, and tail-dominant surfaces first so the fix targets the real bottleneck instead of the loudest symptom.
- Prefer narrow repairs at the dominant cost center over another broad scheduler toggle. If a few giant files are setting the tail, fix scan-kernel or ownership behavior there first, then revisit higher-level traversal doctrine from the lower tail-tax floor.
- Native predecessor parity is a hard performance contract. A suite split near parity, such as 6 wins and 6 losses against the installed previous IX binary, means the investigation is not done even when individual wins are impressive. Use the previous/native binary as the floor to beat, not as a loose comparison target.
- Treat benchmark glare as hostile. Before editing, dive below headline ratios into byte-level workload shape, retained byte volume, candidate filters, regex decomposition, byte charting, match density, thread topology, shard geometry, aggregation cost, and slowest-file tails. The target is the actual cost center, not the most visible branch.
- Never promote assumption-first optimization. When a candidate idea is still unproven, isolate its core mechanism in the smallest removable slice that can be benchmarked against the current canonical binary and the installed native binary. Expand only after repeated interleaved samples show a real average win with match-count parity.
- Research must be specific and demand-driven. If byte charting, regex execution, SIMD/memmem behavior, Aho-Corasick strategy, thread scheduling, cache locality, or Windows filesystem behavior becomes the active unknown, use focused web/Insect research on that exact mechanism before designing the durable fix. Do not perform broad topic surveys as a substitute for profiling the live lane.
- Prefer bottom-up proof over broad rewrites. A quick isolated function, temporary probe, microbench, or telemetry counter is acceptable when it narrows the blast radius and can be removed cleanly. A full implementation without a preceding proof slice is not acceptable for performance work.
- Require enough samples to defeat machine-load noise. One or two faster pairs are not proof. Use interleaved multi-pair comparisons, compare medians and win counts, and rerun suspicious near-ties before calling a lane fixed or lost.
- Keep the 50% faster ambition visible. If improvements flatten, do not keep shaving the same surface. Re-examine the architecture for a qualitatively different path: less scanning, better prefilter rejection, cheaper aggregation, reduced materialization, sharper prepared-input reuse, or different concurrency ownership.
- Once native install is present, prefer `ix` for local search and search-validation workflows. During command-surface migration, repo-local `target/release/ix.exe` is the preferred binary and `target/release/iex-cli.exe` is compatibility-only. Do not use `rg` for local repo search in this workspace unless `ix` is unavailable and that blocker is recorded in evidence.
- Before any rebuild that could replace `target/release/ix.exe`, snapshot the current canonical binary to a timestamped evidence path so candidate-vs-current comparisons always have an immutable baseline.
- Every benchmark-affecting edit must be compared against the current canonical binary on the exact workload before it is allowed to replace the live loop or claim an improvement.
- Live loop promotion is its own proof gate: compare the candidate against the exact binary currently driving the active loop on the full suite with interleaved multi-pair samples, archive the proof artifact, and only then repoint the loop to a timestamped immutable snapshot if the suite-level result shows real gains instead of a one-off lane win.
- Benchmark lanes are contract-specific. Keep the canonical CLI lane (`ix search` via `runOneBenchmark`) separate from the prepared replay lane (`iex-bench` via `runPreparedReplayBenchmark`).
- Prepared replay is an internal steady-state replay metric, not a headline competitor metric. Do not compare prepared replay artifacts directly against `ripgrep`, `iex_previous`, or the canonical CLI lane when making product-speed claims.
- Canonical CLI reports own `tools/reports/live-metrics.jsonl` and `tools/reports/latest.json`. Prepared replay reports own `tools/reports/prepared-live-metrics.jsonl` and `tools/reports/prepared-latest.json`. Never let one lane overwrite the other's report files.
- Non-ASCII case-insensitive regex comparisons require match-count parity before `iex_previous` timing is treated as authoritative. If counts diverge, keep the measurement as timing-only and mark the caveat in the benchmark artifact and console output.
- No backfill and no fallback shortcuts.
- No split-brain architecture.

### Deep Technical Research Standard

When the user asks for deep research, interpret depth as implementation-level file-search knowledge: engine internals, byte scanning, regex execution strategy, automata construction, SIMD/memmem behavior, cache locality, branch behavior, allocation pressure, shard scheduling, reduction geometry, Windows I/O, corpus distribution, benchmark statistics, and source-level precedent from serious search engines. The research bar is below the public headline and inside the mechanism: source code, maintainer issue threads, benchmark harnesses, papers, talks, profiler traces, and failure reports that explain why a path is slow or fast.

Do not flatten "deep" into generic web summaries, and do not reinterpret it as a cybersecurity objective. The target is faster lawful file search. Search language should be technically sharp enough to surface advanced implementation discussions while staying focused on benign performance engineering: exact algorithm names, hot-loop costs, false-positive rates, memory traffic, scheduling overhead, match-count semantics, and workload-specific evidence. Every imported idea must still pass the IX proof loop: smallest removable probe, match-count parity, interleaved current-vs-installed benchmarks, retained only when neutral or faster.

### Performance Candidate Retention Discipline

Every performance edit has a mandatory candidate lifecycle. Baseline first: snapshot `target/release/ix.exe`, capture exact current-vs-installed proof on the affected lane before editing, and confirm match-count parity plus phase shape. Probe second: implement only the smallest removable mechanism slice, then benchmark that candidate against both the pre-change snapshot and the installed native binary. A single win is not promotion evidence, and a single noisy loss is not rejection evidence; near-ties and suspicious reversals require stronger interleaved rechecks before the decision is final.

Retention requires all gates to pass: candidate median beats the current snapshot, candidate is neutral-or-better against installed native, authoritative lanes keep identical match counts, and adjacent known-sensitive lanes do not regress. Rejection is mandatory when repeated current-snapshot proof loses, when telemetry shows the targeted mechanism did not activate, when a shared-owner guard lane regresses, or when the idea requires a broad parallel system instead of a bounded proof slice. A rejected candidate must leave durable evidence: hypothesis, exact code seam, proof artifact, rejection reason, cleanup/revert evidence, and next lower-level mechanism.

Deep iteration does not weaken the gate. Research produces a testable hypothesis, not permission to land. Do not keep drilling the same invalid variant after a stronger recheck fails; archive the negative result and advance to the next ranked mechanism. Worse-than-current candidates are reverted even if they beat the installed native binary once, because the current canonical binary is the promotion floor and the installed binary is the predecessor floor.

## Do / Don't

### Do
- Reverse-engineer best-in-class open source patterns before implementing.
- Keep each change testable and measurable.
- Preserve UX/state lifecycle parity for every surfaced capability.
- Record benchmarks with reproducible commands and objective evidence.
- Keep docs and tests in lockstep with behavior.

### Don't
- Don't introduce hidden side channels or one-off integration paths.
- Don't patch symptoms while leaving root-cause architecture unchanged.
- Don't overengineer beyond current objective.
- Don't degrade readability to chase novelty.

## Why These Rules Exist
- To keep IX fast, reliable, and maintainable under aggressive iteration.
- To prevent architectural drift and duplicate ownership.
- To ensure every capability has a measurable contract.
- To keep delivery quality high without codebase entropy.

## How To Execute
1. Recon first: map source of truth, architecture boundaries, and dependencies.
2. Plan second: write deterministic todo chains before broad implementation.
3. Profile third: identify the lowest true cost layer before editing. Separate scan kernel, byte filtering, regex strategy, shard execution, discovery/materialization, aggregation, and output retention costs.
4. Prove fourth: test the smallest removable mechanism slice against the current canonical binary and the installed native binary. If it loses, revert it and record the rejection before trying another path.
5. Build fifth: implement atomic slices only after the proof slice shows a repeatable win.
6. Verify always: snapshot `target/release/ix.exe` before rebuilding, then run focused owner tests, `cargo test -p iex-core --quiet`, exact candidate-vs-snapshot proof, exact candidate-vs-installed proof, telemetry activation checks, and adjacent-lane guards when ownership is shared before swapping the canonical runner.
7. Promote carefully: before changing the active CLI loop, compare against the currently running loop binary on the full suite, pin the promoted binary to a timestamped immutable snapshot, then confirm `tools/reports/latest.json` is reading that snapshot path after the restart.
8. Keep scenario truth explicit: use the prepared replay lane only for reuse/discovery-isolation proof, and keep its evidence under the prepared report files so it cannot be mistaken for canonical CLI proof.
9. Close cleanly: update docs and preserve evidence.

## Where Rules Apply
- Entire monorepo root and all submodules.
- Rust crates, test harness, benchmark tooling, dashboard, docs, and scripts.
- Any future package added to this workspace.

## Required Skill Sequence
- Start: `user-message-logger`, `recon-intel`.
- Before execution: `planning-spec`.
- UI tasks: `ux-playbook`.
- Deep code tasks: harvest open-source references first.
- Turn closeout: run duplicate-risk review and update crown-jewel documentation.

## Definition Of Done
- Capability implemented end-to-end with no fallback workaround paths.
- Test matrix materialized and passing for changed contracts.
- Benchmark evidence captured with reproducible commands and the timestamped pre-rebuild canonical-binary snapshot path.
- For performance work, the edited binary is measured against the current canonical binary and only promoted when the comparison proves the change is neutral or better on the target workload.
- For native predecessor work, the edited binary is measured against the installed previous IX binary with enough interleaved samples to show median improvement and acceptable win-pair direction; 6/6 suite splits require another investigation slice.
- For deep optimization work, the final handoff names the specific cost center investigated, the proof slice attempted, the artifact proving win or loss, and the next lower-level mechanism to inspect.
- For rejected performance candidates, the final handoff records the hypothesis, code seam, proof artifact, repeated-sample decision, revert evidence, and the next ranked mechanism so the same invalid area is not blindly reopened.
- If the active loop is updated, the promoted binary is an immutable snapshot and the dashboard telemetry is verified against that exact path after restart.
- If prepared replay tooling is touched, prepared replay evidence is captured under the prepared report files and is explicitly kept separate from canonical CLI loop evidence.
- Docs updated with architecture rationale and usage instructions.
- No new duplicate ownership or parallel systems introduced.

---
> Source: [svgop/ix-rust](https://github.com/svgop/ix-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
