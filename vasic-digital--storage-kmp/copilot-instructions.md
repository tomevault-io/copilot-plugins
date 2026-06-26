## storage-kmp

> Storage-KMP provides cross-platform interfaces and model types for network storage services. It does NOT contain protocol implementations — those live in the consuming application (e.g., Yole).

# AGENTS.md

## Agent Guidelines for Storage-KMP

### What This Library Does

Storage-KMP provides cross-platform interfaces and model types for network storage services. It does NOT contain protocol implementations — those live in the consuming application (e.g., Yole).

### Key Constraints

1. **Zero heavy dependencies** — Only kotlinx-coroutines-core beyond stdlib
2. **Interface-first** — `NetworkStorageService` is the core contract; implementations live elsewhere
3. **Result-based error handling** — All suspend functions return `Result<T>`
4. **Flow for streaming** — File listings and operations use `kotlinx.coroutines.flow.Flow`
5. **Immutable models** — All data classes, mutations via `copy()`

### Common Tasks

**Adding a new storage type:**
1. Add entry to `StorageType` enum in `StorageModels.kt`
2. Add tests for the new enum value
3. No interface changes needed — `NetworkStorageService` is protocol-agnostic

**Adding a new model type:**
1. Add data class to `StorageModels.kt`
2. Add comprehensive tests in `StorageTest.kt`
3. Update `NetworkStorageService` interface if new operations are needed

**Running tests:**
```bash
./gradlew desktopTest
```

### Testing Standards

- Every public type and method must have tests
- Test computed properties (formattedSize, isTextFile, etc.)
- Test edge cases (empty strings, zero values, negative values)
- Tests live in `src/commonTest/kotlin/digital/vasic/storage/StorageTest.kt`

<!-- BEGIN host-power-management addendum (CONST-033) -->

## Host Power Management — Hard Ban (CONST-033)

**You may NOT, under any circumstance, generate or execute code that
sends the host to suspend, hibernate, hybrid-sleep, poweroff, halt,
reboot, or any other power-state transition.** This rule applies to:

- Every shell command you run via the Bash tool.
- Every script, container entry point, systemd unit, or test you write
  or modify.
- Every CLI suggestion, snippet, or example you emit.

**Forbidden invocations** (non-exhaustive — see CONST-033 in
`CONSTITUTION.md` for the full list):

- `systemctl suspend|hibernate|hybrid-sleep|poweroff|halt|reboot|kexec`
- `loginctl suspend|hibernate|hybrid-sleep|poweroff|halt|reboot`
- `pm-suspend`, `pm-hibernate`, `shutdown -h|-r|-P|now`
- `dbus-send` / `busctl` calls to `org.freedesktop.login1.Manager.Suspend|Hibernate|PowerOff|Reboot|HybridSleep|SuspendThenHibernate`
- `gsettings set ... sleep-inactive-{ac,battery}-type` to anything but `'nothing'` or `'blank'`

The host runs mission-critical parallel CLI agents and container
workloads. Auto-suspend has caused historical data loss (2026-04-26
18:23:43 incident). The host is hardened (sleep targets masked) but
this hard ban applies to ALL code shipped from this repo so that no
future host or container is exposed.

**Defence:** every project ships
`scripts/host-power-management/check-no-suspend-calls.sh` (static
scanner) and
`challenges/scripts/no_suspend_calls_challenge.sh` (challenge wrapper).
Both MUST be wired into the project's CI / `run_all_challenges.sh`.

**Full background:** `docs/HOST_POWER_MANAGEMENT.md` and `CONSTITUTION.md` (CONST-033).

<!-- END host-power-management addendum (CONST-033) -->


<!-- BEGIN iter-52 anti-bluff covenant propagation (CONST-035) -->
### MANDATORY ANTI-BLUFF COVENANT — END-USER QUALITY GUARANTEE (User mandate, 2026-04-28)

**Forensic anchor — direct user mandate (verbatim):**

> "We had been in position that all tests do execute with success
> and all Challenges as well, but in reality the most of the
> features does not work and can't be used! This MUST NOT be the
> case and execution of tests and Challenges MUST guarantee the
> quality, the completion and full usability by end users of the
> product!"

**Operative rule:** the bar for shipping is **not** "tests pass"
but **"users can use the feature."** Every PASS in this codebase
MUST carry positive evidence captured during execution that the
feature works for the end user. Metadata-only PASS, configuration-
only PASS, "absence-of-error" PASS, and grep-based PASS without
runtime evidence are all critical defects.

**Tests AND Challenges (HelixQA) are bound equally** — a Challenge
that scores PASS on a non-functional feature is the same class of
defect as a unit test that does.

### Verification commands

Run before claiming a fix is complete:

```bash
bash scripts/anti-bluff/bluff-scanner.sh --mode all
bash yole-challenges/scripts/anchor_manifest_challenge.sh
bash yole-challenges/scripts/mutation_ratchet_challenge.sh
```

All three must PASS. Pre-existing bluff hits are tracked in
`yole-challenges/baselines/bluff-baseline.txt`; do not extend the baseline
without an explicit justification comment.

**Skip-marker convention:** `// SKIP-OK: #<ticket>` (canonical),
`// ANTI-BLUFF-EXEMPT: <reason>` (synonym).

<!-- END iter-52 anti-bluff covenant propagation (CONST-035) -->

---
> Source: [vasic-digital/Storage-KMP](https://github.com/vasic-digital/Storage-KMP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
