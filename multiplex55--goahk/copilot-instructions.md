## goahk

> Guidance for AI agents working in this repository.

# AGENTS.md

Guidance for AI agents working in this repository.

## Project identity

`goahk` is a Windows-first automation runtime inspired by AutoHotkey v2, implemented as a Go-native, script-first library.

The primary user experience is writing small Go programs that bind hotkeys to built-in actions or arbitrary Go callbacks. Go is the scripting language. Config files are compatibility plumbing, not the product direction.

Canonical authoring style:

```go
app := goahk.NewApp()

app.Bind("Ctrl+Alt+B", goahk.SendText("basic script trigger"))

app.Bind("Ctrl+Shift+V", goahk.Func(func(ctx *goahk.Context) error {
    text, err := ctx.Clipboard.ReadText()
    if err != nil {
        return err
    }
    return ctx.Input.Paste(strings.ToUpper(text))
}))

app.Bind("Escape", goahk.ControlStop())

if err := app.Run(context.Background()); err != nil {
    log.Fatal(err)
}
```

All new public examples, docs, tests, and APIs should reinforce this model.

## Highest-priority rules

- Treat `goahk.NewApp().Bind(...).Run(...)` as the canonical workflow.
- Prefer Go functions, action helpers, callback services, and fluent builder methods over config/schema changes.
- Do not make JSON, TOML, YAML, or any other config format the primary authoring model.
- Do not introduce a custom scripting language, AHK parser, DSL, daemon, installer, service host, or machine-wide runtime unless explicitly requested.
- Preserve existing config compatibility unless the task explicitly says to change it, but keep compatibility code secondary.
- Keep simple scripts simple. Avoid speculative abstractions that make one-file automation programs harder to write.
- Return clear errors for user-facing script mistakes. Do not panic for validation, binding, hotkey, action, callback, or compatibility-config errors.
- Keep runtime internals independent from the public `goahk` package.
- Keep Windows-specific syscall, COM, unsafe, hook, SendInput, clipboard, UIA, and window code behind internal service boundaries and build tags.
- Do not use GUI/tooling code as the design center. Maintain the automation runtime and script-first API as the stable core.

## Repository map

Important areas:

- `goahk/` — public script-first Go API. This package is the primary product surface.
- `internal/program/` — internal normalized binding/program model and validation.
- `internal/runtime/` — compiler, dispatcher, supervisor, lifecycle, shutdown, control-plane/work-plane behavior.
- `internal/hotkey/` — hotkey parsing, normalization, conflict detection, listener abstractions, Windows registration/listening.
- `internal/actions/` — built-in action registry, execution, action adapters, callback bridge.
- `internal/input/` — keyboard and mouse input services.
- `internal/clipboard/` — clipboard formats, history, watch helpers, Windows/native clipboard access.
- `internal/window/` — active window, matching, enumeration, activation, geometry.
- `internal/uia/` and `internal/inspect/` — UI Automation and inspection backend services.
- `internal/process/` — launching/opening external processes or OS targets.
- `internal/app/` — app/bootstrap/lifecycle/reload wiring.
- `internal/config/` — compatibility adapter only. Do not build new primary UX here.
- `internal/testutil/` — fakes, stubs, golden helpers.
- `cmd/goahk/` — compatibility runner/CLI entrypoint.
- `cmd/goahk-inspect/` — inspection CLI.
- `examples/` — runnable script-first examples.
- `docs/` — architecture, usage, testing, build, ADRs, and code-first documentation.

Ignore GUI implementation details when making core runtime/API decisions. Do not add instructions, tests, or architecture that assume the current GUI shell is permanent.

## Public API design rules

### Shape of the API

New user-facing behavior should normally appear as one of these:

1. A small public action helper in `goahk/`.
2. A method on `*goahk.App` or `*goahk.BindingBuilder`.
3. A callback-facing service or method on `*goahk.Context`.
4. An internal runtime/action/service capability compiled from the public API.

Preferred public style:

```go
app := goahk.NewApp()

app.Bind("Ctrl+Alt+T", goahk.SendText("hello"))

app.On("Ctrl+Shift+R").Replace().Do(goahk.Func(func(ctx *goahk.Context) error {
    for i := 0; i < 10; i++ {
        if err := ctx.Err(); err != nil {
            return err
        }
        ctx.Log("working", "step", i)
        if !ctx.Sleep(250 * time.Millisecond) {
            return ctx.Err()
        }
    }
    return nil
}))
```

Avoid adding public APIs that require users to construct internal model structs directly.

### Naming conventions

Public Go API naming should be concise, idiomatic, and script-friendly:

- Constructors use `NewX` when they create durable values, for example `NewApp`.
- Binding verbs use short imperative names: `Bind`, `On`, `Do`, `Run`.
- Fluent policy methods use PascalCase action words: `Serial`, `Drop`, `QueueOne`, `Parallel`, `Replace`.
- Public action helpers use PascalCase and read like script commands:
  - `SendText`
  - `SendKeys`
  - `SendChord`
  - `Open`
  - `OpenURL`
  - `OpenFolder`
  - `Launch`
  - `StartApplication`
  - `ClipboardRead`
  - `ClipboardWrite`
  - `ClipboardAppend`
  - `ClipboardPrepend`
  - `ActivateWindow`
  - `CopyActiveWindowTitle`
  - `ListOpenApplications`
  - `ListOpenFolders`
  - `MessageBox`
  - `Log`
  - `Stop`
  - `ControlStop`
  - `ControlHardStop`
- Callback wrapper remains `goahk.Func(func(ctx *goahk.Context) error { ... })`.
- Callback context services should remain obvious nouns: `ctx.Input`, `ctx.Clipboard`, `ctx.Window`, `ctx.Process`, `ctx.UIA`, `ctx.Automation`, `ctx.Runtime`, `ctx.AppState`, `ctx.Vars`.
- Use `ctx.Vars` for per-trigger scratch state copied through the current action chain.
- Use `ctx.AppState` for shared process-wide script state.

Internal action IDs may use domain-style names such as `input.send_text`, `clipboard.write`, `window.activate`, or `runtime.control_stop`. Do not leak these internal IDs into examples as the preferred user-facing API.

### Adding a new built-in action

When adding a built-in action:

1. Add a public helper in `goahk/` unless the behavior is internal-only.
2. Compile it into an internal `program.StepSpec` without exposing internal structs to users.
3. Register/execute the internal action in `internal/actions/` or the appropriate service package.
4. Keep parameters typed and simple at the public layer.
5. Add tests for the public helper, compile output, executor behavior, service adapter behavior, and errors.
6. Add a script-first example when the action is broadly useful.
7. Only update config adapter support when compatibility needs it, and add config parity tests if so.

Do not add config-only actions. If the action is useful enough for users, it should be available from Go first.

## Hotkey conventions

Maintain stable, readable hotkey syntax:

- Support chords such as `Ctrl+Alt+T`, `Win+Space`, `CapsLock+A`, `Shift+F1`, and `Escape`.
- Normalize modifier aliases consistently.
- Prefer canonical modifier display order: `Ctrl`, `Alt`, `Shift`, `Win`, then the key.
- Normalize `Esc` to `Escape`.
- Normalize single-letter keys to uppercase.
- Detect duplicates and conflicts with clear errors.
- Prevent repeated firing from held keys unless repeat behavior is explicitly enabled.
- Keep conflict reports actionable enough that a script author can fix the binding quickly.

Do not create config-only condition systems for context-aware hotkeys. Prefer Go callbacks using `ctx.Window`, `ctx.UIA`, `ctx.Clipboard`, `ctx.AppState`, and normal Go conditionals.

## Runtime and concurrency behavior

Keep the runtime deterministic and responsive:

- Startup and shutdown should be deterministic.
- Hotkeys must unregister cleanly during shutdown.
- Normal action execution must not block input processing.
- Control-plane actions must remain separate from work-plane action execution.
- `ControlStop()` and `ControlHardStop()` must stay responsive even if normal callbacks are blocked.
- Long-running callbacks must receive cancellation and should honor `ctx.Err()`, `ctx.Context()`, and `ctx.Sleep(...)`.
- Service calls should be cancellation-aware where practical.
- Logs should make registration, dispatch, cancellation, policy decisions, service failures, and shutdown diagnosable.

Per-binding execution policies must remain available:

- `Serial()` / `serial` — ignore triggers while already running.
- `Drop()` / `drop` — drop triggers while busy.
- `Parallel()` / `parallel` — allow every trigger to run.
- `QueueOne()` / `queue-one` — keep one pending execution.
- `Replace()` / `replace` — cancel the running execution and start the newest one.

Prefer the fluent public methods in examples. String policies via `WithPolicy(...)` are acceptable for compatibility or advanced code, but examples should usually use `.Serial()`, `.Drop()`, `.QueueOne()`, `.Parallel()`, or `.Replace()`.

## Config compatibility policy

`internal/config` and `cmd/goahk` are compatibility surfaces. They are not the primary UX.

Rules:

- Do not add new config schema as the first or only way to use a feature.
- Do not write new docs that lead with JSON config.
- Do not turn examples into config files.
- Do not restructure the runtime around config schemas.
- If existing config behavior changes, preserve parity where possible and add/adjust config adapter tests.
- Config should adapt into the same internal program/runtime model used by the Go API.

Acceptable compatibility work:

- Fixing existing config loading/validation bugs.
- Keeping old config files working.
- Adding config adapter parity for a feature that already exists in the Go API.
- Improving error messages for compatibility mode.

## UI, viewer, and inspection guidance

The automation runtime is the stable core. UIA inspection tools are supporting tools.

- Keep inspection logic in `internal/inspect`, `internal/uia`, or `internal/window`.
- Keep GUI code in command packages.
- Do not move runtime, hotkey, action, input, clipboard, process, or window logic into GUI packages.
- Do not make any GUI framework or generated UI bindings part of core architecture.
- Do not spend effort preserving UI-shell-specific files unless the task explicitly concerns them.
- When replacing or removing GUI code, do it deliberately and keep backend inspection tests intact.

## Build and verification policy

### Do not run automation apps by default

Do not use these as default verification commands:

```powershell
go run ./cmd/goahk
go run ./cmd/goahk -config <file>
go run ./examples/...
```

Those commands can start interactive hotkey listeners, UI apps, or long-running automation processes. Only run interactive commands when the user explicitly asks for manual/runtime verification.

Prefer build and tests.

### Standard checks

Run commands from the repository root.

Minimum expected checks for most Go changes:

```powershell
go mod download
go build -v ./...
go vet ./...
go test -v ./...
```

Run formatting before finalizing Go changes:

```powershell
gofmt -w <changed-go-files>
```

For concurrency, cancellation, supervisor, dispatcher, state, listener, or hotkey-manager changes, also run:

```powershell
go test -race ./...
```

If full-repo commands fail because unrelated GUI code is in transition, do not treat that as a core-runtime failure. Run focused package checks for the changed packages and clearly report the limitation.

### Windows integration tests

Default tests should be deterministic and non-interactive. Real Windows input/hotkey behavior requires an interactive Windows desktop session and should be gated.

Use integration-tagged tests only when appropriate:

```powershell
go test -tags=integration ./internal/runtime ./internal/hotkey
```

For real keyboard/mouse injection checks, only enable the required environment variable when explicitly requested:

```powershell
$env:GOAHK_ENABLE_WINDOWS_INPUT_ITEST = "1"
go test -tags=integration ./internal/runtime ./internal/hotkey
```

Manual UI/hotkey testing must not be the only validation for a code change.

## Test expectations

Add or update tests for behavior changes. Prefer focused package tests over broad, brittle integration tests.

Expected coverage areas:

- Public API happy paths and error messages.
- Public examples and snippets that compile.
- Hotkey parsing, normalization, alias handling, and conflict detection.
- Binding validation and duplicate/conflict errors.
- Program normalization and compile behavior.
- Action dispatch ordering.
- Built-in action parameter mapping and executor behavior.
- Callback execution, cancellation, logging, vars/state propagation, and service access.
- Per-binding concurrency policy behavior.
- Control-plane vs work-plane behavior.
- Clipboard/window/input/process/UIA adapters using fakes or stubs.
- Windows-specific implementations behind build tags.
- Config adapter parity only when compatibility behavior changes.
- Golden fixtures when command output or serialized behavior changes.

Use `internal/testutil` fakes where possible. Normal unit tests must not require real clipboard state, real window focus, real keyboard input, real global hotkeys, or a live UI Automation tree.

## Documentation expectations

When changing public behavior, update docs in the same change:

- `README.md` for primary quick-start or public API changes.
- `docs/USAGE.md` for user-facing usage details.
- `docs/architecture.md` for runtime, lifecycle, service, or package boundary changes.
- `docs/testing.md` for test strategy changes.
- `docs/BUILD.md` for build command or toolchain changes.
- `docs/adr/` for durable architectural decisions.
- `examples/` for runnable code-first demonstrations.

Docs and examples should lead with normal Go programs. Compatibility config can be documented separately, but it must not be presented as the default way to author automation.

Good example topics:

- Basic hotkey sending text.
- Clipboard transform and paste.
- Window-aware behavior using Go conditionals.
- Launching apps and opening folders/URLs.
- Emergency stop hotkeys with `ControlStop`.
- Long-running callback with `.Replace()` and cancellation checks.
- UIA selector usage from Go callbacks.

## Error handling and logging

- Return errors with actionable operation context.
- Wrap lower-level errors where it improves diagnosis.
- Do not swallow errors in listener, registration, dispatch, service, callback, or shutdown paths.
- Avoid logging sensitive clipboard contents by default.
- Keep log fields structured and stable when tests assert on them.
- Prefer explicit validation errors over silent no-op behavior.

## Dependency and artifact policy

- Keep `go.mod` aligned with the repository's supported Go version.
- Avoid dependencies for simple standard-library functionality.
- Prefer small, focused dependencies only when they materially improve Windows/runtime reliability.
- Do not commit generated binaries or build artifacts: `.exe`, `.dll`, `.so`, `.dylib`, `bin/`, coverage outputs, temporary zips, or generated UI assets.
- Do not hand-edit generated files when generation is still used by a command package.
- Keep OS-specific code isolated behind interfaces and build tags.

## Implementation style

- Keep changes small and focused.
- Preserve existing package boundaries.
- Prefer interfaces at OS/service boundaries, not broad speculative abstraction through every layer.
- Keep public API names concise and Go-idiomatic.
- Do not expose internal model types unless there is a clear user-facing reason.
- Preserve backward compatibility unless explicitly told otherwise.
- Prefer deterministic tests over timing-heavy tests. When timing is unavoidable, use fakes, fake clocks, or bounded timeouts.
- Avoid global mutable state unless it is explicitly part of app/runtime state and tested for concurrency safety.
- Keep callback state explicit through `ctx.Vars` and `ctx.AppState`.

## Before final response

When reporting work back to the user, include:

- Files changed.
- Why the changes were made.
- Verification commands run.
- Any commands that could not be run and why.
- Any follow-up risk, especially around Windows-only behavior or integration tests that require an interactive desktop.

---
> Source: [multiplex55/goahk](https://github.com/multiplex55/goahk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
