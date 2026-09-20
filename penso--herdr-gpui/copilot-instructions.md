## herdr-gpui

> Engineering guidance for agents working in this repository. Read this alongside

# AGENTS.md

Engineering guidance for agents working in this repository. Read this alongside
`README.md`, the relevant crate documentation, and the code before making changes.

## Priorities

- Keep changes small, explicit, and focused on the root cause.
- Preserve user and other agents' changes. Never revert unrelated edits.
- Prefer existing standard traits and generics over unnecessarily concrete APIs.
- Protect protocol compatibility, bounded resource use, and UI responsiveness.
- Add regression tests for changed behavior and report what was actually verified.

## Architecture

Herdr GPUI is a native client of an existing local Herdr daemon, not a terminal
emulator, server, or TUI wrapper. Herdr owns terminal processes and session state.
Closing or detaching the GUI must leave the daemon and its terminals running.

| Location | Responsibility |
| --- | --- |
| `crates/herdr-protocol` | Generation-1 wire types, framing, validation, and atomic surface patches |
| `crates/herdr-client` | Discovery, socket worker, session transitions, ordered commands, and client-local activity projection |
| `crates/herdr-gpui` | Native UI, connection bridge, presentation state, semantic input, geometry, and painting |
| `crates/test-support/sandbox.rs` | Shared isolated process setup for opt-in integration tests; not a production crate |

- Keep dependency direction from UI to client to protocol. Protocol/client code must not depend on GPUI.
- Keep session transitions separate from socket scheduling, and connection ownership separate from window rendering.
- Reuse `ConnectionBridge`, domain targets, geometry helpers, and the test sandbox rather than duplicating their policies.
- Split modules by responsibility, not arbitrary line counts. Prefer wiring and exports in entry points as code grows; do not perform unrelated file reshuffles.
- Keep APIs narrow. Use private items by default and `pub(super)` or `pub(crate)` only where needed; do not expose every field merely to ease extraction.
- Define shared types once in the lowest appropriate layer and re-export them. Do not create mirror enums or convert between them through strings.
- Do not add a crate, runtime, service container, or speculative transport abstraction without a concrete need. Sibling checkouts are references, not build dependencies.

## Rust Conventions

- Require only the capabilities an operation uses. Prefer existing traits such as `Read`, `Write`, `IntoIterator`, `AsRef<Path>`, and `AsRef<OsStr>` over specific streams, collections, or owned paths.
- Use slices, `&str`, and `&Path` when borrowing is sufficient. Keep owned data where it must cross a thread boundary or outlive its caller.
- Prefer `impl Trait` or named generics for static dispatch. Use `dyn Trait` for actual runtime heterogeneity/type erasure, not by default.
- Use `FnOnce`, `FnMut`, or `Fn` for injectable behavior. Do not invent a callback or clock trait when a closure or explicit time value suffices.
- Do not introduce custom traits merely to wrap one struct. A new trait needs a concrete behavioral contract and a real implementation/substitution need that existing traits cannot express.
- Use enums for closed alternatives such as navigation targets, pane/popup input destinations, connection phases, and launch modes. Avoid string dispatch and boolean-plus-ID pairs internally.
- Parse strings at boundaries, then match on domain types. Prefer `From`, `TryFrom`, `Into`, and `TryInto` for conversions.
- Prefer typed request/result structures when shapes are known. Preserve genuinely open-ended protocol envelopes rather than forcing a speculative schema.
- Derive standard traits such as `Default`, `PartialEq`, and `Eq` where their semantics are valid. Compare complete values when deduplicating operations.
- Propagate errors with `?`; retain actionable categories and source/context until the display boundary. Implement meaningful `Display` and `Error`, not debug-only user messages. Do not add error-framework dependencies for trivial wrapping.
- No `unwrap()` or `expect()` in production. Test-only allowances must be scoped to test code. Keep unsafe exceptions narrow and document their safety conditions.
- Prefer guard clauses and readable iterators. Avoid clones, allocations, helper layers, and generic parameters that provide no benefit.
- Comments should explain invariants, ownership, or non-obvious decisions, not narrate assignments.

## GPUI Rules

- Treat app, window, and entity contexts as UI-thread execution. A foreground `spawn` future does not make blocking work safe.
- Never perform blocking socket/disk I/O, process waits, sleeps, or expensive background computation on the UI thread. Use the existing worker threads or GPUI background executor, then apply results through the appropriate context update.
- Keep render and input paths thin. Render from prepared state and bounded caches; do not query the daemon, scan files, or launch processes while rendering.
- Preserve continuously drained ordered events and the coalesced latest-state mailbox. Do not replace them with an unbounded UI event queue.
- Route connection/startup failures through the authoritative inbox. Reconnect and detach must isolate old inboxes so delayed events and paint acknowledgements cannot affect the new connection.
- Derive connection text/indicators from `ConnectionStatus`. Old local operation errors must not hide a disconnection reason.
- Queue success is not daemon acknowledgement. Keep names such as `last_queued_options` honest and include cell pixel metrics in resize comparisons.
- Use the same geometry for painting, hit testing, and IME placement. Popup input and composition bounds must target the popup, not the underlying pane.
- Keep glyph visibility separate from decorations: spaces and wide-character continuation cells may still need underline/strikethrough painting.
- Preserve menu input isolation, focus behavior, Unicode composition, and semantic input routing. Do not synthesize terminal keys for operations with endpoint API methods.
- Activity indicators come from protocol state-change sequences, not guessed terminal output. Acknowledge completion only for the coherent surface actually presented in an active window, with the existing boot/revision/sequence fences.

## Protocol And Safety

- Vendored bincode field and enum variant order is a compatibility contract. Do not reorder declarations as cleanup. Preserve attribution in `crates/herdr-protocol/NOTICE.md`.
- Keep frame/response/geometry bounds, strict decoding, and atomic patch validation. Test malformed inputs and limits, not only valid fixtures.
- Never mix snapshot metadata and surface cells across boot IDs or projection revisions. Preserve future-surface buffering and stale-boot rejection.
- Preserve the single in-flight API request, bounded command/event queues, FIFO ordering, bounded write batches, and request correlation.
- Partial frame prefixes/payloads must survive read timeouts. Finish an inbound partial frame before dispatching commands against potentially stale state.
- Cancellation must not flush queued work, replay commands, or join a worker on the UI thread. Do not add automatic reconnect/replay as an incidental refactor.
- Crossbeam receiver clones compete for events; they are not broadcast subscribers.
- Treat terminal content and daemon messages as untrusted data. Do not execute terminal escapes, follow graphics file paths, modify the clipboard, or run displayed update commands automatically.
- The normal app never installs, starts, stops, or upgrades a personal daemon. Explicit sockets refer to the binary client socket, not the JSON API socket.
- Never commit secrets, credentials, private terminal output, or machine-local configuration.

## Dependencies And Features

- Put dependency versions in root `[workspace.dependencies]`; member crates inherit with `{ workspace = true }`.
- Use the pinned `rust-toolchain.toml` and GPUI version. Do not copy toolchain, feature, or release assumptions from Moltis/Arbor.
- Keep workspace lint inheritance and fix warnings rather than widening allowances.
- Keep `integration-test` opt-in. Gate imports, modules, and call sites consistently, and verify both default and all-feature builds.
- Do not introduce Tokio or another runtime to wrap the existing blocking socket worker.
- Avoid unrelated lockfile upgrades. Use `--locked` for reproducible verification.

## Verification

Prefer the existing `just` recipes. Run focused tests while iterating, then the
workspace gates for Rust changes before handoff or a requested commit:

```sh
just format
just ci
```

`just ci` runs these checks:

```sh
cargo fmt --all -- --check
cargo clippy --locked --workspace --all-targets --all-features -- -D warnings
cargo test --locked --workspace
cargo test --locked --workspace --all-features
```

- For linking, startup, or packaging changes, also use `just test-build` to build the release executable and exercise its CLI without a desktop.
- Documentation-only changes need command/path/link review and `git diff --check`; do not claim a code test run that did not happen.
- Add deterministic tests for invariants: ordering, cancellation, fragmentation, revision fencing, error paths, geometry, and cleanup. Prefer explicit coordination and bounded waits over timing guesses.
- Test pure session/CLI/geometry logic without a socket or window where possible. Exercise public behavior with mock peers and headless GPUI tests where integration matters.
- Fix flaky tests rather than hiding them with retries or new ignores. Existing live/native tests are ignored because they require explicit external resources.
- For visual changes, verify the actual native UI when a desktop is available, including narrow layouts, long labels, focus, popup/IME behavior, and clipping. Headless layout tests do not prove native glyph or OS input correctness.
- Measure interactive performance in release mode (`just run`), not debug mode. Preserve bounded glyph caching and deterministic paint budgets; native timing budgets are machine-dependent.
- Keep CLI parsing pure and based on OS strings so socket paths need not be UTF-8. Invalid options and help must exit before starting GPUI.

### Opt-In Tests

```sh
just test-live /absolute/path/to/herdr
just test-gui /absolute/path/to/herdr
just test-sidebar
just test-perf
```

- Live tests require an explicitly selected `HERDR_TEST_BINARY`; never fall back to discovering a user's daemon.
- Reuse the shared sandbox: private unique short directory, cleared environment, isolated HOME/XDG/config/socket paths, and explicit desktop-variable allowlist for GUI children only.
- Kill and wait only for exact children created by the test, before removing their sandbox. Never use `server stop`, process-name matching, or process-group kills.
- `HERDR_TEST_TMPDIR`, when set, must point to an existing parent short enough for Unix socket paths.
- Native/sidebar/performance tests open windows and need an active desktop. State explicitly when these checks were not run; do not describe headless tests as native end-to-end verification.

## Git And Handoff

- Only commit, push, or create/update a PR when requested. Do not amend, force-push, discard changes, or clean up other worktrees without explicit authorization.
- Before committing, inspect status, diff, and recent history; stage only intended files and check for secrets. Do not bypass hooks or signing when they fail.
- Use conventional commit subjects (`feat`, `fix`, `refactor`, `test`, `docs`, `chore`). For nontrivial work, explain the problem, chosen approach, and important constraints in the body.
- Do not add AI attribution trailers or assistant-session links unless explicitly requested.
- Before creating a PR, review every included commit and the full diff from its base. Include a summary, exact validation commands/results, and remaining manual/native QA.
- Keep user-facing docs and public API examples synchronized with behavior changes. Do not copy another repository's release, issue-tracker, or mandatory-push workflow into this one.
- Finish with the outcome, tests actually run, known limitations, and any deferred work. Never claim unrun checks passed.

---
> Source: [penso/herdr-gpui](https://github.com/penso/herdr-gpui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
