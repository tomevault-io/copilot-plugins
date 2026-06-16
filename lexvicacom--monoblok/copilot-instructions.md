## monoblok

> Guidance for coding agents working in this C/libuv monoblok tree.

# AGENTS.md

Guidance for coding agents working in this C/libuv monoblok tree.

## Shape

Monoblok is a compact C17 NATS-like daemon with a patchbay routing DSL,
optional LVC, snapshots, JSON helpers, NATS bridge/core import, and
consumer-only JetStream import through vendored `nats.c`.

Keep the layout shallow:

- `src/`: core daemon pieces (`array`, `buf`, `fs`, `proto`, `router`,
  `slice`, `snapshot`, `bridge`, `importer`, `jetstream`, `nats_common`,
  `main`).
- `src/server/`: libuv listener, connection lifetime, read/write callbacks.
- `src/patchbay/`: arenas, S-expression/JSON/YAML adapters, program loading,
  evaluator dispatch, form fragments, validation, soundcheck, and dump tooling.
- `test/`: unit tests, NATS fakes, CLI checks, smoke/soundcheck scripts, and
  load/soak checks.
- `examples/`: runnable patchbay examples and demo configs.
- `advanced-examples/`: larger integration examples and producer/pump scripts.
- `docs/`: user-facing overview, patchbay docs, and documentation images.
- `design/`: design notes and proposals that are not necessarily product docs.
- `scripts/`: benchmark, smoke, packaging, install, service, and release helpers.
- `seccomp/`: container seccomp profile.
- `.github/workflows/`: CI and release automation.
- `.vscode/`: shared workspace editor settings.
- `bench-results/`: checked-in benchmark result notes.
- `vendor/libuv/`: vendored libuv source.
- `vendor/nats.c/`: vendored NATS C client.
- `vendor/yyjson/`: pruned yyjson source, license, and readme.
- Root files: CMake entrypoint, Dockerfile, readmes/agent notes, format/git
  config, and the default `patchbay.edn`.
- `build*/` and `dist/`: local/generated build and package output, not source
  layout.

Prefer local, explicit C over frameworks or abstraction layers. The point of
this branch is proving plain C can stay readable while staying fast.

## Project taste

Monoblok is close in spirit to the Redis Manifesto: a small daemon exposing a
clear data/protocol model, with memory-first predictable behavior and complexity
kept visible instead of hidden behind opaque layers.

- Treat the protocol, router, LVC, snapshots, and patchbay as concrete data
  structures with visible cost. APIs should make the model easier to reason
  about, not pretend hard tradeoffs disappeared.
- Prefer in-memory state and bounded, predictable hot paths. Disk is for
  snapshots and warm start, not a second storage engine unless the project
  explicitly changes shape.
- Say no to features that require broad abstraction, background magic,
  publish-time allocation, or large dependency surfaces for a marginal win.
- Dependencies are acceptable when they are self-contained, audited, and keep
  the main story smaller; vendored code should remain an island.
- Code should be pleasant to read because ownership, lifetimes, and performance
  consequences are obvious. Favor direct, local C over clever generality.

## Safety and correctness

Monoblok tends not to live directly on the public internet, but correctness
still matters. Bad input, accidental misuse, and future deployment changes can
find the same bugs as a hostile client.

- Be mindful of buffer overruns, integer overflow, unterminated data, borrowed
  slice lifetimes, and unchecked lengths whether the trigger looks nefarious or
  ordinary.
- Favor explicit logic over clever logic. Do not overcomplicate control flow,
  lifetime rules, or error handling for a marginal reduction in lines.
- Write simple, boring, descriptive C that a human can reason about under
  pressure. Prefer names that describe the domain object or state transition.
- Do not over-abstract. Keep helpers local until there is a real repeated
  domain concept.
- Do abstract common domain-specific operations behind project helpers with
  `mb_` names, such as file-open/read helpers. Memory helpers should make
  ownership obvious with `mb_<thing>_alloc` and `mb_<thing>_free` names.
- Keep load-bearing project structs briefly commented with their role,
  ownership, or lifetime. Do not comment for the sake of commentary; add
  comments when code is fiddly, protocol behavior is non-obvious, or an
  invariant would otherwise be easy to miss.
- Be open about repetitive work. Load tests, soak tests, and smoke tests should
  be automated where possible; editing repetitive comments by hand is often not
  productive.

## Build and test

```sh
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
ctest --test-dir build --output-on-failure
cmake --build build --target smoke
cmake --build build --target soundcheck
scripts/bridge-smoke.sh
scripts/import-smoke.sh
scripts/jetstream-smoke.sh
```

Sanitizer pass:

```sh
cmake -S . -B build-asan -DMB_ASAN=ON
cmake --build build-asan
ctest --test-dir build-asan --output-on-failure
```

Useful targets:

```sh
cmake --build build --target monoblok
cmake --build build --target bench-patchbay
cmake --build build --target pb-dump
```

## Core invariants

- The server runs on one libuv loop thread. Do not add hot-path mutexes or
  atomics unless the threading model changes.
- Keep one guarded `uv_write_t` per connection. Queue bytes in buffers, swap
  into `in_flight`, and resubmit only after completion.
- Router owns subscriptions, LVC, bridge callback hooks, and output fanout only.
  It must not know about libuv handles.
- Server owns connection lifetime and write completions.
- Protocol parsing is slice-based and allocation-free.
- `mb_slice`/`pb_slice` values are borrowed views unless a helper explicitly
  duplicates them. Do not store a slice beyond the lifetime of its source
  buffer or arena.
- Patchbay parse trees and temporary eval values live in arenas.
- Long-lived patchbay state is owned by `pb_eval_state`; be explicit about
  string/ring ownership and free every heap field in `state_entry_free`.
- Avoid surprise publish-time allocation. Allocate on connection open, SUB,
  UNSUB, patchbay/config load, bridge/import startup, or first state-slot
  creation; reuse read/write buffers and per-publish scratch arenas. Import
  copies remote messages into a bounded cross-thread ring and reuses slot
  buffers after growth.
- JetStream import is pull-based patchbay ingress. Startup catch-up runs before
  the local listener opens, live fetch happens on worker threads, and evaluation
  is handed back to the libuv loop one message at a time. Keep that handoff
  backpressured; do not let JetStream worker threads mutate router or evaluator
  state directly.
- On Linux, libuv uses epoll for the event loop. The daemon disables libuv's
  optional io_uring paths by default for seccomp-friendly containers; use
  `--io-uring` or `UV_USE_IO_URING=1` to opt in before loop creation.

## Patchbay model

- Lists are call forms. A list must be non-empty and headed by a symbol.
- Vectors are data.
- JSON patchbay files use arrays as call forms and `{"vec":[...]}` for vector
  data. Plain objects in form arguments expand to keyword/value pairs.
- Do not add quote, macro, or list-as-data compatibility paths.
- Keep effectful forms explicit, conventionally with `!` (`publish!`,
  `json-demux!`).
- Before adding a new DSL form, read `src/patchbay/README.md` and keep the
  evaluator/module split in sync with that guidance. Once the DSL reaches
  critical mass, be disciplined about saying no; do not turn patchbay into an
  ill-conceived Swiss Army knife of marginal forms.
- If an evaluator form is not implemented, validation should fail rather than
  silently accepting it.
- Patchbay messages re-enter rule evaluation only when the emitting rule is
  marked `:reentrant true`. Re-entry is capped to avoid infinite loops.
- Negative numeric literals and negative payloads are valid. When testing with
  `nats pub`, pass negative bodies after `--`, for example
  `nats pub sensors.temp -- -5`.

## When porting .edn patchbays to .yml

- Read `docs/patchbay-yaml-schema.md` first; it is the canonical YAML shape
  reference. Use `docs/patchbay.md#yaml-sugar-files` for the narrative
  overview and `docs/patchbay-cheatsheet.md#edn--json--yaml` for the short
  EDN/JSON/YAML comparison.
- Treat YAML as a small sugar layer that lowers to the same patchbay AST, not
  as general YAML. The root is a map with `lvc`, `on`, `export`, deprecated
  `bridge`, and/or `import`; each `on` item is a rule map with `sub` and one
  body shape such as `thread`, `when`, `do`, `form`, or `body`.
- Use flow arrays for call forms, with the operator as the first element. Use
  `thread` for `->`, `when` for `(when ...)`, and `form`/`body` for direct
  expressions such as `transition`, `count!`, or `bar!`.
- Be precise with scalars. In expression positions only `subject`, `payload`,
  `payload-float`, and `payload-int` become bound symbols; other string-like
  scalars become strings unless they are numbers, booleans, nulls, or keywords
  such as `:ms`. Quote subjects, URLs, and payload strings when it improves
  clarity or avoids YAML punctuation traps.
- In top-level config string positions, `env: NAME` lowers to `(env "NAME")`.
  Do not use it inside rule bodies. Validate the port with `monoblok
  --validate` and, when behavior matters, compare with `--soundcheck`.

## LVC and snapshots

`$LVC.<subject>` is a live last-value stream. Subscribing registers a normal sub
against the stripped inner subject with an `is_lvc` flag, and immediately emits
matching cached values. Publishing to `$LVC.*` is rejected. `--no-lvc` disables
the cache and rejects `$LVC.*` subscribes.

Snapshots are optional warm-start persistence:

- `--snapshot PATH`: load at startup if present.
- `--snapshot-every SECONDS`: periodic dump.

Snapshots include LVC entries and patchbay state. Snapshot rule state identity
is `(rule_idx, filter)`. If the patchbay changed and the recorded filter no
longer matches, that rule state is skipped with a warning. LVC entries load
regardless.

## Bridge and import mode

The outbound bridge is optional and export-only. It is configured by a
top-level `(export ...)` form in the patchbay file. Deprecated `(bridge ...)`
is still accepted as a compatibility alias. Leaving the form out makes runtime
cost zero.

Bridge fan-out uses `Router.bridge_fn`, called once per publish after local
delivery. The bridge does its own subject-filter matching and publishes matching
subjects to the remote NATS cluster through `nats.c`, which owns reconnects and
outbound buffering.

Import mode is optional inbound tap mode. It is configured by a top-level
`(import ...)` form. `:core` entries subscribe to remote NATS subjects through
`nats.c`, copy messages into a bounded ring, and wake the libuv loop for
patchbay evaluation. `:streams` entries consume JetStream durable pull
consumers and feed matching messages into patchbay only.

Imported raw messages are private patchbay ingress and must not be routed to
direct monoblok subscribers unless a rule republishes them explicitly. When
`(import ...)` is configured, local socket clients may still subscribe but
client `PUB` commands are rejected.

JetStream import is consumer-only; monoblok is not a JetStream server. Each v1
stream entry has one subject filter plus `:stream`, `:consumer`, and optional
`:catch-up`. With `:catch-up true`, startup replay runs to the stream's
startup high-water before the local listener opens, using JetStream message
metadata as event time. During that phase rules see `replaying?` as true; after
catch-up, the same durable consumers continue in live mode with `replaying?`
false. Ack only after the patchbay handler and downstream publish path succeed;
malformed subjects, oversized payloads, missing metadata, eval failure, or
publish failure should leave the JetStream message unacked for redelivery.

## NATS protocol scope

Core local server only: `CONNECT`, `PUB`/`MSG` reply-to, `SUB`, `UNSUB`,
`PING`, `PONG`, `INFO`, `+OK`, `-ERR`.

There is no auth on the local server side, headers, JetStream service, or
`$SYS.*` service request-reply. JetStream support is import-side only through
configured `:streams` durable consumers. The bridge remains export-only and
does not route remote replies back. Import mode consumes remote NATS and
JetStream messages as patchbay input only. `CONNECT` bodies are accepted and
ignored. `+OK` is never sent.

## C style

- Project-owned code currently targets C17. The root CMake config sets
  `CMAKE_C_STANDARD 17`, requires that standard, and disables compiler
  extensions. Do not introduce C23 constructs, compiler-specific extensions,
  non-portable tricks, or exotic language features without asking first and
  making a serious case for the tradeoff.
- The root `.clang-format` intentionally uses `ColumnLimit: 0` to avoid
  save-on-format churn in generated-looking compact C. Do not run broad
  mechanical formatting unless explicitly requested.
- Keep every project-owned struct documented with one short comment explaining
  its role, ownership, or lifetime. This includes small view types such as
  slices.
- Prefer arena copies for AST text and evaluator-owned output text.
- Keep comments concise and focused on ownership, lifetime, invariants, or
  non-obvious protocol behavior.
- Do not let vendored code inherit project warning flags.
- Only suggest third-party libraries when they fit the project shape: small,
  self-contained, easy to audit, compatible with the build, and justified by
  removing more complexity than they add. Prefer vendoring small C libraries
  over `FetchContent` or package-manager magic, so builds remain predictable
  and dependency code stays reviewable.
- `pb_eval.c` contains evaluator dispatch and binding semantics. Keep large
  builtin bodies in `pb_builtins.c` or split them further before the evaluator
  gets hard to scan.

## Platform Assumptions

Linux is the primary deployment target. Development primarily happens on macOS.

Aim to write vanilla, portable C and boring shell scripts where practical. Be
aware of the small but annoying portability traps between Linux and Darwin,
especially:

- BSD vs GNU `grep`, `sed`, `awk`, `readlink`, `date`, and other shell utility
  variants.
- Darwin C runtime and libc behavior differences.
- Filesystem, socket, signal, and process-edge cases that may behave differently
  across platforms.
- Compiler and linker flag differences between Apple Clang and Linux toolchains.

Do not fix a macOS inconvenience by making Linux worse, and do not introduce
Linux-only assumptions into generic code unless the deployment path explicitly
justifies it.

Windows support may matter one day, but it is not a current target. Do not add
Windows abstraction layers, compatibility scaffolding, or build complexity unless
explicitly asked.


## Verification

For most source changes, run:

```sh
cmake --build build
ctest --test-dir build --output-on-failure
```

For behavior touching I/O, routing, snapshots, bridge, or import, add the relevant
smoke test:

```sh
cmake --build build --target smoke
scripts/bridge-smoke.sh
scripts/import-smoke.sh
scripts/jetstream-smoke.sh
```

Use a `Release` CMake build before reporting performance.

---
> Source: [lexvicacom/monoblok](https://github.com/lexvicacom/monoblok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
