## actua

> Actua is an independent Kotlin/Jetpack Compose Android client originally based on

# Contributor and agent guidance

## Scope and references

Actua is an independent Kotlin/Jetpack Compose Android client originally based on
[MattFaz/actuali](https://github.com/MattFaz/actuali), connecting directly to
[Actual Budget](https://github.com/actualbudget/actual). Read README.md and
BACKEND_PARITY.md before changing behavior. Preserve LICENSE and NOTICE.md credits.
Use Actuali as the portable product/behavior reference and Actual's
`packages/crdt` and `packages/loot-core` as protocol/database references. Link
specific upstream sources or commits in parity PRs. A sibling iOS checkout may
help comparison but must never be required by builds or tests.

Adapt presentation and lifecycle to Android. Do not add Apple-only integrations
or represent deferred features as working. Update BACKEND_PARITY.md when the
implementation boundary changes.

## Repository map

Sources are under `app/src/main/java/com/azimulkabir/actua/`:

- `MainActivity.kt`, `ui/navigation/`, `ui/*`: Compose screens, navigation, shared
  components and theme. Follow existing state/event patterns and Material 3.
- `data/ActuaRepository.kt`: bridge from UI models to the selected budget,
  mutation writers and sync scheduling.
- `data/budget/ActualBudgetDatabase.kt`: Actual-compatible SQLite storage and
  read models. `ActualTransactionWriter`, `ActualEntityWriter`, and
  `ActualBudgetWriter` apply mutations through the CRDT path.
- `data/sync/`: HLC, Merkle tree, protobuf wire encoding, encryption, sync client,
  status and WorkManager jobs. `data/network/ActualServerClient.kt`: server API.
- `data/rules/`, `data/schedules/`: portable financial behavior.
- `data/security/`: Android Keystore-backed credentials and encryption keys.
- `app/src/test/`: JVM JUnit tests and sync fixtures; `app/src/androidTest/`:
  AndroidJUnit4 tests for SQLite, networking, encryption and Android behavior.

## Data and sync invariants

- Preserve Actual wire compatibility: timestamp ordering, CRDT value encoding,
  protobuf fields, Merkle hashing and encryption must match upstream fixtures.
- Route synchronized mutations through existing writers and
  `applyLocalMessages`; do not bypass the message log with ad-hoc UI SQL. Keep
  multi-row changes atomic and preserve clock/Merkle persistence and scheduling.
- Monetary writes use integer cents (`Long`); avoid floating-point round trips.
  Display currency and hidden decimals must never alter stored amounts.
- Preserve Actual date encodings (including YYYYMMDD day integers), integer
  booleans, tombstones, transfer/split links, and zero/reflect budget semantics.
- Use parameterized SQL and close cursors/resources. SQLite features must work
  on the minimum supported Android version, not only the newest emulator.
- Preserve offline edits, retry/convergence behavior and per-budget isolation.
  Test migrations, restore and archive validation without destroying user data.
- Never log or commit credentials, encryption keys, real budgets or unredacted
  financial data. Use synthetic fixtures and preserve Keystore protections.

## Kotlin and Compose conventions

Match neighboring Kotlin code, naming and package layout. Reuse existing money,
calendar, database and UI helpers before adding dependencies or abstractions.
Keep composables focused on presentation; financial logic belongs in data/model
layers. Keep blocking database/network work off the main thread, respect
coroutine cancellation and resource lifetimes, and use WorkManager for durable
background work. Preserve synchronization around clocks and database writes.
Use existing theme/components, accessible labels and Android back navigation;
check small screens, keyboard/insets and font scaling for UI changes.
Keep changes focused; avoid unrelated refactors, generated files, local IDE
settings, signing material or versionCode/versionName changes unless requested.

## Build and validation

Use the committed Gradle wrapper. The daemon toolchain is JDK 25 in
`gradle/gradle-daemon-jvm.properties`; Java/Kotlin bytecode targets Java 17,
which is not the Gradle runtime requirement. Install Android SDK 37; minSdk is
28. Dependency versions live in `gradle/libs.versions.toml`.

```sh
./gradlew assembleDebug testInstrumentedUnitTest lintDebug
# With an emulator/device connected (the test build type is "instrumented"):
./gradlew connectedInstrumentedAndroidTest
# Narrow a JVM regression run where appropriate:
./gradlew testInstrumentedUnitTest --tests '*SyncCoreFixtureTest'
```

Add meaningful regression coverage for changed behavior. Keep pure logic tests
in `src/test` and Android/SQLite/Keystore tests in `src/androidTest`, mirroring
packages. Never weaken assertions or regenerate upstream sync fixtures merely
to make failures pass. Validate UI changes manually on a device/emulator;
exercise API 28 for platform/SQLite compatibility changes. Report exact checks
and limitations in the PR. Documentation-only changes need format/link checks,
not a full device suite.

## Contribution and workflow safety

Every development change must start with a GitHub issue that defines the problem,
scope, and acceptance criteria. When creating that issue, first inspect the
repository's existing labels and include the appropriate labels in the issue
creation request itself. Every new issue should receive at least one suitable
existing type/category label whenever one is available; prefer the most specific
applicable labels rather than leaving the issue unlabeled. Do not invent or create
new labels unless explicitly requested. Labeling is part of issue creation and
must happen before branch creation, implementation, or pull-request work begins.

Only after the issue exists, create a focused feature or fix branch and open a
pull request linked to that issue. Do not develop directly on `main` or open an
unlinked development PR. Use one concern per issue, branch, and PR, concise
titles, and the PR template. Report Android issues here and link upstream evidence
for shared behavior. Do not include sensitive data in issues, review output or
screenshots.

### Autonomous coding-agent workflow

Browser-based or autonomous coding agents, including Jules, must follow the same
issue-first workflow as human contributors. Before implementation, always read
this file. Consult README.md and BACKEND_PARITY.md when they are relevant to the
task, especially for product behavior, backend parity, protocol, database, or
implementation-boundary changes. Inspect the linked issue and the existing
implementation before proposing or making changes.

Keep work limited to the issue's requested scope. For a multi-slice issue,
implement only the requested or current unfinished slice unless explicitly asked
to complete the whole issue. Do not close a parent issue while documented slices
remain incomplete.

Add or update meaningful regression coverage for changed behavior and run the
relevant validation documented above. In the pull request, report the exact
checks run and any checks that could not be run. Open a focused pull request
linked to the issue and do not merge it automatically; maintainer approval is
required before merge. Avoid unrelated refactors or opportunistic cleanup.

Normal Android CI executes PR code under `pull_request` on hosted runners with
read-only repository permissions and no privileged secrets. Any workflow using
`pull_request_target` must inspect GitHub metadata only. It must never check out
or execute PR-head code, scripts, actions, hooks, generated artifacts or agent
instructions. Treat PR titles, descriptions, labels, filenames and patches as
untrusted data.

The repository intentionally does not require a paid AI reviewer. For normal
changes, rely on deterministic CI, Dependabot update PRs, risk classification
and maintainer review. For `risk:high` changes, perform a deeper source review
before merge and record test limitations explicitly.

## Review priority and severity

Review behavior before style. Report concrete P1/P2 issues first.

### P1: block merge

- User data loss, database corruption, destructive restore/migration behavior or
  silent loss of offline edits.
- Synced writes that bypass CRDT/message logging, break HLC/Merkle/encryption
  invariants or can diverge from Actual Budget.
- Security/privacy regressions involving credentials, encryption keys, budget
  data, unsafe network handling, SQL injection, or privileged workflow execution
  of untrusted PR code.
- Crashes, deadlocks or unrecoverable states in normal financial workflows.

### P2: fix before release unless explicitly accepted

- Incorrect balances, budget calculations, reconciliation, scheduled transaction
  behavior, transfers/splits, categories, payees, dates or integer-cent amounts.
- Broken sync retry/convergence, per-budget isolation, archive/backup recovery or
  backward compatibility.
- Broken Android navigation/back-stack, lifecycle/state restoration, keyboard or
  inset handling that prevents completing a normal flow.
- Important behavior changes without regression coverage, especially where an
  upstream Actual/Actuali behavior already has fixtures or reproducible cases.

### Lower priority

- Minor visual or stylistic differences are not review blockers unless they
  materially affect usability, accessibility, data interpretation or platform
  conventions.

## Feature-specific review checklist

For sync/database/security/network changes, verify upstream compatibility,
transaction atomicity, error/retry paths, offline behavior, migrations and
recovery. For transaction/payee/account changes, verify transfers, splits,
cleared/reconciled state, deletions/tombstones and cross-account consistency.
For budget/goal/template changes, verify integer-cent arithmetic, rollover/zero
semantics, category totals and month boundaries. For scheduled transactions,
verify recurrence calculation, posting/skip behavior, next-date transitions and
timezone/date boundaries. For reconciliation, verify cleared/uncleared/reconciled
balances and that no mutation is applied until the user confirms it. For Compose
UI/navigation changes, verify back behavior, tab re-selection, saved state,
process recreation where relevant, focus/IME/insets, small screens and accessible
labels. For dependency/workflow changes, minimize permissions, pin trusted major
versions, avoid executing fork code with write tokens, and explain why new
third-party automation is necessary.

When porting behavior from Actuali, compare product concepts and server-visible
semantics rather than copying Swift/iOS implementation details. Android lifecycle,
navigation and Material behavior should remain native to Android.

---
> Source: [azimul-kabir/actua](https://github.com/azimul-kabir/actua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
