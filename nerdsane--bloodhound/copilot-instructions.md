## bloodhound

> Instructions for AI agents and human developers working on Bloodhound.

# Bloodhound Development Guide

Instructions for AI agents and human developers working on Bloodhound.

## Project Overview

Bloodhound is a **deterministic simulation testing platform** for hunting bugs in distributed systems using a modified QEMU hypervisor.

**Key Goals:**
- Perfect reproducibility: Same seed = identical execution, always
- Language agnostic: Test any containerized application without modification
- Systematic fault injection: BUGGIFY-style deterministic faults
- Time-travel debugging: Full replay capability with GDB integration

## Engineering Philosophy

**Three pillars ground all code in Bloodhound:**

1. **[TigerStyle](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md)** - Coding standards (assertions, explicit limits, naming)
2. **Formal Specifications** (`specs/tla/`, `src/stateright/`) - Protocol correctness
3. **Deterministic Simulation Testing** - Fault tolerance verification

**Before writing code, consult all three.** They define what "correct" means.

Priority order: **Safety > Performance > Developer Experience**

### The Verification Pyramid

All three pillars are **connected through shared invariants** (`src/invariants/`):

```
         TLA+ Specs (specs/tla/*.tla)
                    │ mirrors
         Shared Invariants (src/invariants/*.rs)  ← SINGLE SOURCE OF TRUTH
                    │ used by
    ┌───────────────┼───────────────┐
    ▼               ▼               ▼
Stateright      DST Tests      Production Code
(exhaustive)    (simulation)   (TigerStyle)
```

**Each invariant has:**
- `name` - matches TLA+ property name
- `tla_spec` - source spec file reference
- `tla_line` - line number in spec
- `holds` / `violation` - result with context

**Example:** When implementing snapshot tree operations:
1. Read `specs/tla/SnapshotTree.tla` for the protocol
2. Use `SnapshotPropertyChecker` from `invariants/snapshot.rs` in your tests
3. Properties like `no_orphan_snapshots` are verified identically across all layers

### Core Principles

1. **Determinism Above All**: Every execution with the same seed must produce identical results. Non-determinism is a bug.

2. **Simulation-First**: Test infrastructure comes before implementation. If you can't test it deterministically, don't build it.

3. **Fault Injection is Not Optional**: Distributed systems fail in complex ways. We inject faults systematically, not as an afterthought.

4. **Honest Documentation**: Document what works AND what doesn't. Never claim something is tested if it isn't.

## Quick Rules

### Assertions (2+ per function)
```rust
use crate::tigerstyle::{assert_precondition, assert_postcondition, assert_invariant};

fn transfer(from: &mut Account, to: &mut Account, amount: u64) -> Result<()> {
    // Preconditions: What must be true before
    assert_precondition!(amount > 0, "amount must be positive");
    assert_precondition!(from.balance >= amount, "insufficient balance");

    let old_from_balance = from.balance;
    from.balance -= amount;
    to.balance += amount;

    // Postconditions: What must be true after
    assert_postcondition!(from.balance < old_from_balance, "from balance must decrease");

    Ok(())
}
```

### Explicit Limits (bound everything)
```rust
const SNAPSHOT_SIZE_BYTES_MAX: usize = 1024 * 1024 * 1024;  // 1GB
const VM_COUNT_MAX: usize = 64;
const SIMULATION_STEPS_MAX: u64 = 1_000_000;
const FAULT_PROBABILITY_MAX: f64 = 0.5;

if snapshot_size > SNAPSHOT_SIZE_BYTES_MAX {
    return Err(Error::LimitExceeded(...));
}
```

### Big-Endian Naming (most significant first)
```rust
// GOOD
snapshot_size_bytes_max
simulation_time_ms
fault_probability_default

// BAD
max_snapshot_size
simulationTimeMs
```

### Deterministic RNG Usage
```rust
// GOOD: Use FaultActor's seeded RNG
let should_fault = fault_actor.maybe_fault("disk_write", 0.01);

// BAD: Non-deterministic random
let should_fault = rand::random::<f64>() < 0.01;
```

### Error Handling
```rust
// GOOD: Typed errors with context
return Err(SimulationError::VmFailed {
    vm_id,
    reason: "hypercall timeout".into(),
    seed: self.seed,
});

// BAD: String errors
return Err("VM failed".into());
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                       BLOODHOUND                            │
│              (Modified QEMU Hypervisor)                     │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Virtual Time │  │ Fault Inject │  │ State Snap   │      │
│  │ (TSC, HPET)  │  │ (Net, Disk)  │  │ (CoW, Tree)  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
├─────────────────────────────────────────────────────────────┤
│                    GUEST VMs (Containers)                   │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐           │
│  │  web   │  │ redis  │  │postgres│  │  ...   │           │
│  └────────┘  └────────┘  └────────┘  └────────┘           │
└─────────────────────────────────────────────────────────────┘
```

### Actor Hierarchy

```
SimulationCoordinator
├── TimeActor (owns virtual clock, deterministic time)
├── FaultActor (schedules/injects faults deterministically)
├── EventCollectorActor (trace recording)
├── WorkloadActor (deterministic operation generation)
└── VmActors[] (one per VM)
    ├── Vm (QEMU wrapper or simulated)
    ├── HypercallChannel
    └── Fault injection state
```

## Module Structure

```
src/
├── actor/                  # Actor implementations
│   ├── time_actor.rs       # Virtual clock control
│   ├── fault_actor.rs      # Fault scheduling, RNG
│   ├── event_collector.rs  # Event tracing
│   ├── workload_actor.rs   # Operation generation
│   └── vm_actor.rs         # VM lifecycle wrapper
├── buggify/                # BUGGIFY fault injection macros
├── config.rs               # Configuration parsing (YAML)
├── container/              # Docker/container integration
├── error.rs                # Typed error system
├── explore/                # State space exploration
├── fault/                  # Fault types and deterministic RNG
│   └── deterministic_rng.rs
├── harness/                # Simulated VM for fast testing
│   └── simulated_vm.rs
├── hypervisor/             # QEMU integration
│   ├── qemu.rs             # QMP client
│   └── qmp.rs              # QMP protocol
├── invariants/             # Shared invariant definitions (NEW)
│   ├── mod.rs              # PropertyResult, PropertyChecker trait
│   ├── snapshot.rs         # Snapshot tree invariants
│   ├── determinism.rs      # Determinism verification
│   └── fault.rs            # Fault injection correctness
├── lib.rs                  # Library exports
├── main.rs                 # CLI entry point
├── simulation/             # Core simulation logic
│   ├── coordinator.rs      # Main orchestration
│   └── state.rs            # Snapshot tree
├── stateright/             # Exhaustive model checking (NEW)
│   ├── snapshot_model.rs   # Snapshot tree state machine
│   └── fault_model.rs      # Fault scheduling model
├── tigerstyle/             # Assertion macros
├── web/                    # Web UI (optional)
└── workload/               # Workload drivers

specs/
└── tla/                    # TLA+ formal specifications (NEW)
    ├── SnapshotTree.tla    # Snapshot tree consistency
    ├── DeterministicTime.tla # Virtual time properties
    └── FaultScheduling.tla # Fault injection protocol
```

## Key Files

| File | Purpose |
|------|---------|
| `src/simulation/coordinator.rs` | Main simulation orchestration |
| `src/actor/time_actor.rs` | Deterministic time control |
| `src/actor/fault_actor.rs` | Fault scheduling and injection |
| `src/fault/deterministic_rng.rs` | ChaCha20-based seeded RNG |
| `src/hypervisor/qemu.rs` | QEMU QMP integration |
| `src/harness/simulated_vm.rs` | In-memory VM simulation |
| `src/buggify/mod.rs` | BUGGIFY macro implementation |
| `src/tigerstyle/mod.rs` | Assertion macros |
| `src/invariants/mod.rs` | Shared invariants (PropertyResult, PropertyChecker) |
| `src/invariants/snapshot.rs` | Snapshot tree property checkers |
| `src/stateright/snapshot_model.rs` | Stateright exhaustive model for snapshots |
| `src/config.rs` | Configuration and CLI |
| `src/error.rs` | Typed error definitions |
| `specs/tla/SnapshotTree.tla` | TLA+ spec for snapshot tree correctness |
| `tests/regression_harness.rs` | Deterministic scenario tests |

## Constants Reference

All limits are defined with explicit bounds:

```rust
// VM limits
VM_COUNT_MAX = 64
VM_MEMORY_BYTES_MAX = 8 GB
VM_VCPU_COUNT_MAX = 1  // Single vCPU for determinism

// Simulation limits
SIMULATION_STEPS_MAX = 1_000_000
SIMULATION_TIME_MS_MAX = 5 * 60 * 1000  // 5 minutes
SNAPSHOT_COUNT_MAX = 10_000

// Fault limits
FAULT_PROBABILITY_MAX = 0.5
FAULT_DELAY_MS_MAX = 10_000
PARTITION_DURATION_MS_MAX = 60_000

// Network limits
PACKET_SIZE_BYTES_MAX = 65535
NETWORK_DELAY_MS_MAX = 1000
```

## Common Tasks

### Adding a new fault type
1. Add variant to `FaultType` enum in `src/fault/mod.rs`
2. Add injection logic in `src/actor/fault_actor.rs`
3. Add QEMU command in `bloodhound-ctrl` if hypervisor-level
4. Add configuration parsing in `src/config.rs`
5. Add tests in `tests/regression_harness.rs`

### Adding a new actor
1. Create actor file in `src/actor/`
2. Define message types and handler
3. Register in `SimulationCoordinator`
4. Add unit tests for actor in isolation
5. Add integration tests for actor interactions

### Adding a new workload driver
1. Create driver in `src/workload/`
2. Implement `WorkloadDriver` trait
3. Add configuration support in `src/config.rs`
4. Add example in `examples/`

### Modifying QEMU integration
1. Update patch in `qemu-patches/`
2. Update QMP client in `src/hypervisor/qmp.rs`
3. Update `QemuVm` in `src/hypervisor/qemu.rs`
4. Test with async-VM mode
5. Update `docs/QEMU_FORK_SPECIFICATION.md`

## Testing

```bash
# Unit tests (fast, no QEMU)
cargo test --lib

# Integration tests (some require QEMU)
cargo test --test regression_harness
cargo test --test actor_vm_integration

# Full CI check
cargo fmt --all -- --check
cargo clippy --all-features -- -D warnings
cargo test --all-features

# Run with specific seed for reproduction
SEED=12345 cargo test

# Harness mode (simulated VMs, fast)
cargo run --release -- test --actor-mode --compose docker-compose.yml

# Async-VM mode (real QEMU, thorough)
cargo run --release -- explore --async-vm --compose docker-compose.yml \
  --qemu qemu/build/qemu-system-x86_64 \
  --kernel guest/build/bzImage \
  --initrd guest/build/initramfs.cpio.gz
```

### Test Categories

- **Unit tests**: `src/*/tests/` - Fast, isolated component tests
- **Regression tests**: `tests/regression_harness.rs` - Deterministic scenario tests
- **Integration tests**: `tests/actor_vm_integration.rs` - Multi-component tests
- **Stateright tests**: `src/stateright/` - Exhaustive model checking (run on-demand)

## Formal Verification

Bloodhound uses **formal verification** as a core engineering philosophy. Before implementing complex protocols, we specify them formally.

### TLA+ Specifications (`specs/tla/`)

Human-readable formal specifications for design review and documentation:

| Spec | Models | Key Invariants |
|------|--------|----------------|
| `SnapshotTree.tla` | Copy-on-write snapshot tree | No orphan snapshots, parent-before-child ordering |
| `DeterministicTime.tla` | Virtual clock protocol | Monotonic time, bounded drift |
| `FaultScheduling.tla` | Fault injection scheduling | Reproducible fault ordering, seed consistency |
| `VmLifecycle.tla` | VM state machine | No zombie VMs, clean shutdown |

### Shared Invariants (`src/invariants/`)

**The bridge between specs and code.** Invariants are defined once and used everywhere:

```rust
use crate::invariants::{SnapshotPropertyChecker, PropertyChecker};

// In Stateright model, DST test, or production code
let checker = SnapshotPropertyChecker::new(&snapshot_tree);
let result = checker.no_orphan_snapshots();  // PropertyResult

if !result.holds {
    println!("{}", result);
    // ✗ no_orphan_snapshots (SnapshotTree.tla:142): Snapshot 42 has no parent
}
```

**PropertyResult structure:**
```rust
pub struct PropertyResult {
    pub name: &'static str,      // Matches TLA+ property name
    pub holds: bool,
    pub violation: Option<String>,
    pub tla_spec: &'static str,  // Source spec file reference
    pub tla_line: u32,           // Line number in spec
}
```

| Module | Properties |
|--------|------------|
| `invariants/snapshot.rs` | `no_orphan_snapshots`, `parent_before_child`, `no_duplicate_ids` |
| `invariants/determinism.rs` | `same_seed_same_result`, `time_monotonic` |
| `invariants/fault.rs` | `fault_ordering_deterministic`, `no_fault_without_trigger` |

### Stateright Model Checking

Rust-native exhaustive state space exploration:

```bash
# Run Stateright model checking (on-demand, can be slow)
cargo test stateright -- --ignored --nocapture
```

**Benefits:**
- Runs in CI with `cargo test`
- Uses Rust type system
- **Uses shared invariants** - same property checks as DST tests
- No separate TLC toolchain needed

**Example Stateright model:**
```rust
// src/stateright/snapshot_model.rs
impl Model for SnapshotTreeModel {
    fn properties(&self) -> Vec<Property<Self>> {
        vec![
            // Uses shared invariant from src/invariants/snapshot.rs
            Property::<Self>::always("no_orphan_snapshots", |_, state| {
                let checker = SnapshotPropertyChecker::new(state);
                checker.no_orphan_snapshots().holds
            }),
        ]
    }
}
```

### Kani Bounded Model Checking

**Kani proofs verify that TigerStyle assertions hold for ALL inputs**, not just test cases.

**Install Kani:**
```bash
cargo install --locked kani-verifier
kani setup
```

**Run Kani proofs:**
```bash
# All proofs
cargo kani

# Specific module
cargo kani --harness verify_snapshot_tree_invariants
```

**Kani proofs are in source files under `#[cfg(kani)]` blocks:**

```rust
#[cfg(kani)]
mod kani_proofs {
    use super::*;

    /// Proof: No orphan snapshots (SnapshotTree.tla:142)
    #[kani::proof]
    #[kani::unwind(5)]
    fn verify_snapshot_tree_invariants() {
        let parent_id: u64 = kani::any();
        let child_id: u64 = kani::any();
        kani::assume(child_id > parent_id);

        let mut tree = SnapshotTree::new();
        tree.create_snapshot(parent_id);
        tree.create_child(parent_id, child_id);

        // Must hold for ALL valid inputs
        kani::assert!(tree.get(child_id).parent == Some(parent_id));
    }
}
```

**When to add Kani proofs:**
- State transitions (snapshot create/delete)
- Invariant-preserving operations
- Boundary conditions (limits, thresholds)
- Data structure consistency

**Platform note:** Kani works best on x86_64 Linux. Run in CI for reliable results.

## Verification and Skepticism

**Be skeptical of results until thoroughly verified.** Do not declare something "working" or "production-ready" prematurely.

### Critical: Test Through the Full Production Path

**NEVER claim a feature works unless tested through the actual backend:**

| What You're Testing | Required Backend | NOT Acceptable |
|---------------------|------------------|----------------|
| VM determinism | QEMU TCG mode | Harness mode alone |
| Syscall faults | gVisor DST | Simulated storage |
| Thread races | Hermit | Single-threaded test |
| Full system | Async-VM mode | Harness mode |

**What "tested through production path" means:**
- Start the actual hypervisor/runtime (QEMU, gVisor, Hermit)
- Run with realistic fault injection enabled
- Verify determinism with multiple seeds
- Check that replay produces identical results

### Red Flags to Watch For

- Tests that only use `--backend simulated` when real backends are available
- "It works" based on one seed without testing multiple seeds
- Ignoring warnings about non-determinism in logs
- Testing fault injection with `fault_probability: 0.0`
- Declaring success without verifying replay produces identical trace

### When Reporting Status

- **Bad**: "Determinism works" (after testing only in harness mode)
- **Good**: "Determinism verified in harness mode. QEMU async-VM mode shows 3 non-determinism sources that need fixing (TSC drift, network timing, disk ordering)."

Be precise about what was tested and what limitations remain.

### Verification Requirements

Before claiming a feature works:
1. **Test with actual backend**, not simulated
2. **Verify with multiple seeds** (at least 3 different seeds)
3. **Check replay** produces identical trace
4. **Test with faults enabled** (not just happy path)
5. **Review logs for warnings** about non-determinism

## Benchmarks

### Performance Targets

| Benchmark | What it measures | Target |
|-----------|------------------|--------|
| `snapshot_create` | Snapshot creation latency | <10ms |
| `snapshot_restore` | Restore from snapshot | <50ms |
| `fault_injection` | Fault decision overhead | <1µs |
| `deterministic_rng` | RNG generation speed | >10M ops/sec |
| `event_recording` | Trace event overhead | <100ns/event |
| `vm_step` | Single simulation step | <1ms |

### Running Benchmarks

```bash
# Run all benchmarks
cargo bench

# Compare against baseline
cargo bench -- --save-baseline before
# ... make changes ...
cargo bench -- --baseline before
```

## Checklist Before Committing

### Code Quality (TigerStyle)
- [ ] 2+ assertions per new function (preconditions/postconditions)
- [ ] All limits have explicit constants with `_MAX` suffix
- [ ] Names use big-endian format with units (`size_bytes_max`)
- [ ] Using seeded RNG from FaultActor, not `rand::random()`
- [ ] All errors are typed (no `String` errors)
- [ ] Functions under 70 lines
- [ ] No `HashMap` iteration (use `BTreeMap` for determinism)
- [ ] No `Instant::now()` (use virtual time from TimeActor)

### Verification (NEW)
- [ ] Shared invariants used for any new correctness property
- [ ] Stateright model updated if state machine changed
- [ ] Kani proof added for critical state transitions
- [ ] TLA+ spec updated for protocol changes

### Testing
- [ ] Tests pass with multiple seeds (at least 3)
- [ ] Tested with actual backend (not just simulated)
- [ ] Replay verified to produce identical trace
- [ ] Faults tested (not just happy path)
- [ ] Documentation explains "why", not just "what"

## Architecture Decision Records (ADRs)

ADRs in `docs/adr/` are **living decision logs** that track architectural decisions as they evolve. They are not static documents - update them as you work.

### When to Update an ADR

**During development**, add a Decision Log entry when you:
- Choose between alternative implementations
- Discover constraints that affect the architecture
- Defer or reject a planned feature
- Change approach based on learnings
- Make trade-offs during implementation

**After commits**, update Implementation Status when:
- New components are implemented
- Features are validated/tested
- Items move from "Not Yet Implemented" to "Implemented"

### ADR Quick Reference

| ADR | Update When... |
|-----|---------------|
| `001-deterministic-simulation-architecture.md` | Core DST approach, hypervisor design changes |
| `002-actor-based-design.md` | Actor hierarchy, message types, coordination changes |
| `003-qemu-hypervisor-integration.md` | QEMU patches, bloodhound-ctrl protocol changes |
| `004-fault-injection-strategy.md` | Fault types, BUGGIFY usage, probability handling |
| `005-execution-modes.md` | Harness vs async-VM vs gVisor DST mode changes |
| `006-property-checking-system.md` | Property checks, triggers, StateQueryClient, PropertyExecutor |
| `007-container-translator-caching.md` | Container auto-translation, ImageCache, digest-based caching |
| `008-oci-image-integration.md` | OCI image support, container image handling |
| `009-gvisor-dst-integration.md` | gVisor fork, syscall faults, VirtualClocks, FD filtering |

### Creating a New ADR

When making a decision that doesn't fit existing ADRs:
1. Copy `docs/adr/000-template.md` to `docs/adr/NNN-descriptive-title.md`
2. Fill in Context, Decision, Consequences
3. Add initial Decision Log entry
4. Update `docs/adr/README.md` index

### ADR Supplements

Supplements in `docs/adr/supplements/` contain detailed implementation plans, roadmaps, and metrics that support ADRs but change more frequently.

**When to create a supplement:**
- Detailed implementation roadmaps
- Test results and metrics that get updated over time
- Function/feature catalogs that grow incrementally
- Migration guides or compatibility matrices

**Naming convention:** `NNN-adr-title-supplement-name.md`

### Planning with ADRs

**All implementation plans must be documented in ADRs or supplements.** This ensures:
- Plans are version-controlled and reviewable
- Context is preserved across sessions
- Multiple contributors can understand the approach

**Never leave plans only in session memory** - always persist to ADRs/supplements.

## Three Modes of Operation

1. **Harness Mode** (`--actor-mode`): Simulated VMs, fast, no QEMU required. Use for development and CI.

2. **Async-VM Mode** (`--async-vm --actor-mode`): Actual QEMU VMs with deterministic execution. Use for final validation.

3. **gVisor DST Mode**: Modified gVisor runtime with virtual time and syscall-level fault injection. Use for container-native testing without VM overhead.

Harness and Async-VM modes use the same `SimulationCoordinator`. gVisor DST mode uses a separate implementation in the gVisor fork at `./gvisor/`.

### gVisor DST Mode Details

```
Docker → containerd → runsc-dst (gVisor fork)
                          ↓
                    DST Coordinator
                    ├── VirtualClocks (deterministic time)
                    ├── FaultInjector (syscall-level faults)
                    └── SnapshotTree (state management)
```

Key files in gVisor fork:
- `pkg/sentry/dst/bloodhound.go` - Core DST coordinator
- `pkg/sentry/time/virtual_clocks.go` - Virtual time implementation
- `runsc/config/dst.go` - DST configuration
- `runsc/boot/dst.go` - DST RPC handlers
- `runsc/boot/loader.go` - DST initialization (lines 610-650)

## Backend Fork Locations

All forks are maintained in the `nerdsane` GitHub organization and cloned locally under `/home/sesh/Development/`.

| Backend | Upstream | Fork | Local Path | Build Command |
|---------|----------|------|------------|---------------|
| **Hermit** | [https://github.com/facebookexperimental/hermit](https://github.com/facebookexperimental/hermit) | [https://github.com/nerdsane/hermit](https://github.com/nerdsane/hermit) | `/home/sesh/Development/hermit` | `cargo build --release` |
| **gVisor DST** | [https://github.com/google/gvisor](https://github.com/google/gvisor) | [https://github.com/nerdsane/gvisor-dst](https://github.com/nerdsane/gvisor-dst) | `/home/sesh/Development/gvisor` | `bazelisk build //runsc:runsc --config=x86_64` |
| **QEMU** | [https://github.com/qemu/qemu](https://github.com/qemu/qemu) | [https://github.com/nerdsane/qemu](https://github.com/nerdsane/qemu) | `/home/sesh/Development/bloodhound/qemu` | `./configure --target-list=x86_64-softmmu && make` |

### Binary Locations

After building, binaries are available at:

| Backend | Binary Path | Docker Runtime |
|---------|-------------|----------------|
| **Hermit** | `/home/sesh/Development/hermit/target/release/hermit` | N/A (run directly) |
| **gVisor DST** | `/home/sesh/Development/gvisor/bazel-bin/runsc/runsc_/runsc` | `runsc-dst` |
| **QEMU** | `/home/sesh/Development/bloodhound/qemu/build/qemu-system-x86_64` | N/A (VM hypervisor) |

### Installation Commands

```bash
# Install gVisor DST runtime (requires sudo)
sudo cp /home/sesh/Development/gvisor/bazel-bin/runsc/runsc_/runsc /usr/local/bin/runsc-dst

# Hermit doesn't need installation - run directly with sudo for PMU access
sudo /home/sesh/Development/hermit/target/release/hermit run --seed=42 ./program

# QEMU doesn't need installation - run from build directory
/home/sesh/Development/bloodhound/qemu/build/qemu-system-x86_64 ...
```

### Docker Runtime Configuration

gVisor DST runtime is configured in `/etc/docker/daemon.json`:

```json
{
  "runtimes": {
    "runsc-dst": {
      "path": "/usr/local/bin/runsc-dst",
      "runtimeArgs": [
        "--dst",
        "--dst-seed=42",
        "--root=/var/run/runsc-dst",
        "--dst-control-socket=/tmp/bloodhound-dst/control.sock",
        "--dst-fault-disk-write=0.3",
        "--dst-fault-disk-read=0.3",
        "--dst-fault-network-drop=0.3",
        "--debug",
        "--debug-log=/tmp/gvisor-debug.log"
      ]
    }
  }
}
```

After modifying daemon.json: `sudo systemctl restart docker`

## Common Pitfalls

### Non-Determinism Sources

Watch out for:
- `HashMap` iteration order (use `BTreeMap` or sort)
- Thread scheduling (use single-threaded runtime or deterministic scheduler)
- System time (`Instant::now()`) - use virtual time
- Random numbers - use seeded RNG from `FaultActor`
- File system ordering - sort directory listings
- Floating point operations - may vary by platform

### Testing Mistakes

- Testing only the happy path
- Not testing fault injection
- Assuming harness mode behavior = async-VM behavior
- Not running tests with multiple seeds
- Not documenting what's tested vs untested

## Systematic Debugging Approach

When debugging complex multi-process systems like gVisor DST integration, follow this systematic approach to avoid circular debugging:

### 1. Add Logging at Boundaries First

Before changing logic, add logging at data flow boundaries:
```go
// Log what values are being SET
log.Warningf("SetProbabilities: DiskWrite=%.4f DiskRead=%.4f", probs.DiskWriteFailure, probs.DiskReadFailure)

// Log what values are being READ/USED
log.Warningf("ShouldInjectFault: %s prob=%.4f", faultType, probability)
```

### 2. Trace Data Flow Through Process Boundaries

In multi-process systems, config values often get lost at process boundaries:
```
Parent Process (runsc) → [flags/config] → Child Process (sandbox/boot)
```

Key questions:
- Is the value being set in the config struct?
- Is ToFlags() propagating the value to child processes?
- Is the child parsing the flag correctly?

### 3. Binary Search the Pipeline

When a value shows 0 but should be non-zero:
1. Log at the SOURCE (config parsing)
2. Log at the DESTINATION (where value is used)
3. If source is correct but destination is wrong, binary search the middle

### 4. Document Each Finding

Track what you learn:
```
Config shows: DiskRead=0.2 ✓
SetProbabilities receives: DiskRead=0.0 ✗
→ Problem is between config and SetProbabilities
→ Check ToFlags() propagation
```

### 5. Example: gVisor DST Flag Propagation Bug

**Symptom**: `DiskReadFailure` was 0 despite config having 0.2

**Debug process**:
1. Added logging to `SetProbabilities` → saw `DiskRead=0.0`
2. Verified daemon.json had `--dst-fault-disk-read=0.2` ✓
3. Checked `config/dst.go` flag registration ✓
4. Checked `config/flags.go` `ToFlags()` → **missing `FaultDiskRead` propagation**

**Root cause**: `ToFlags()` propagated `FaultDiskWrite` to child processes but not `FaultDiskRead`

**Fix**: Added missing line in `ToFlags()`:
```go
if c.DST.FaultDiskRead > 0 {
    rv = append(rv, fmt.Sprintf("--dst-fault-disk-read=%f", c.DST.FaultDiskRead))
}
```

### 6. gVisor-Specific: Process Hierarchy

```
docker run → containerd → runsc create/start
                              ↓
                         gofer process (file system)
                              ↓
                         sandbox process (kernel) ← DST runs here
```

Config must flow through ALL of these. Check `ToFlags()` for any new config fields.

## Platform Notes

### Linux (Primary Platform)
- Full support for QEMU/KVM
- All features tested
- Use for final validation

### macOS (Secondary Platform)
- Harness mode works fully
- Async-VM mode is experimental (needs HVF)
- Unit tests pass, but don't assume VM behavior is identical

## AI Collaboration Notes

This project is co-developed with Claude (Anthropic). When working on this codebase:

1. **Be honest** about what's tested vs. untested
2. **Don't over-engineer** - solve the problem at hand
3. **Test everything** - if it's not tested, it's broken
4. **Document decisions** - update ADRs as you work
5. **Ask questions** - clarify requirements before implementing

The goal is to build reliable software, not to impress with clever code.

### Testing Requirements

**When asked to test, always do FULL testing, not what's easy:**

1. **Use the actual backends** - Don't default to `--backend simulated` when Hermit, gVisor DST, or QEMU are available. Check what's installed:
   ```bash
   # Check Hermit
   ls /home/sesh/Development/hermit/target/release/hermit

   # Check gVisor DST
   docker info | grep runsc-dst

   # Check QEMU
   which qemu-system-x86_64
   ```

2. **Test with the appropriate backend for each scenario:**
   - Thread races → Hermit (requires sudo for PMU access)
   - Disk/syscall faults → gVisor DST with fault probabilities configured
   - Full determinism → QEMU TCG mode
   - Fast iteration only → Simulated

3. **Verify determinism** - Run twice with the same seed and confirm identical output

4. **Don't take shortcuts** - If a test requires sudo or special setup, say so and attempt it rather than silently falling back to simulated mode

5. **Report honestly** - If a backend isn't working or configured, say so explicitly rather than pretending the test passed

## Production Observability (Planned)

**Goal:** Close the verification feedback loop by monitoring invariants in production.

### Planned Implementation

When deploying Bloodhound as a service:

1. **Invariant Metrics** (Datadog/Prometheus):
   - `bloodhound.invariant.checks.total` (counter)
   - `bloodhound.invariant.checks.passed` (counter)
   - `bloodhound.invariant.checks.failed` (counter) - **alert on this**
   - `bloodhound.invariant.health` (gauge: 1.0 = healthy)

2. **Log Violations with Context**:
   ```
   ERROR invariant=no_orphan_snapshots tla_spec=SnapshotTree.tla:142
         violation="Snapshot 42 has no parent" seed=12345
   ```

3. **Alerts**:
   - `invariant.checks.failed > 0` → Page on-call
   - `invariant.health < 1` for 5 minutes → Warning

### The Complete Verification Feedback Loop

```
TLA+ Specs → Stateright → Kani → DST Tests → Production → Observability
     ↑                                                          │
     └──────────────── Feedback improves specs ─────────────────┘
```

Formal verification proves properties hold *in theory*. Production observability proves they hold *in practice*. Together, they create a complete feedback loop for generating correct code.

## References

- [TigerBeetle TIGER_STYLE.md](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md)
- [TigerBeetle Safety](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/concepts/safety.md)
- [FoundationDB Testing](https://apple.github.io/foundationdb/testing.html)
- [Antithesis](https://antithesis.com/)
- [Jepsen](https://jepsen.io/)
- [QEMU QMP Protocol](https://wiki.qemu.org/Documentation/QMP)
- [Stateright Model Checker](https://github.com/stateright/stateright)
- [Kani Rust Verifier](https://github.com/model-checking/kani)
- [TLA+ Tools](https://lamport.azurewebsites.net/tla/tools.html)

---

*Last updated: v0.5.0 - Added verification pyramid, shared invariants, Stateright, Kani, and skepticism sections*

---
> Source: [nerdsane/bloodhound](https://github.com/nerdsane/bloodhound) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
