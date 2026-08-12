## anytype-agent-runtime

> A JavaScript agent runtime for Anytype, built on [Sobek](https://github.com/grafana/sobek) (Go JS engine with ES module support).

# Anytype Agent Runtime

A JavaScript agent runtime for Anytype, built on [Sobek](https://github.com/grafana/sobek) (Go JS engine with ES module support).

## Project Structure

```
├── main.go                    # CLI entrypoint: go run . <script.js>
├── anyruntime/
│   ├── anyruntime.go          # Runtime factory, env loading, module chaining
│   ├── anytype_loader.go      # Module loader that fetches JS from Anytype API
│   └── prettyprinter.go       # Trace pretty-printing for CLI output
├── runtime/
│   ├── runtime.go             # Runtime interface, EvalResult type
│   ├── sobek.go               # Sobek-based Runtime implementation, js.eval
│   ├── trace.go               # Trace and TraceRecord types
│   └── hostfn/
│       ├── fetch.go           # Host function: synchronous HTTP fetch
│       ├── chatreply.go       # Host function: chat message output
│       ├── cosine.go          # Host function: native cosine similarity
│       └── sleep.go           # Host function: sleep
├── cmd/chat/                  # Interactive chat CLI
└── utils/
    └── utils.go               # Shared JSON serialization helper
```

## Architecture

### Runtime (`runtime/`)

- **`Runtime` interface** -- `EvalToString`, `SetModuleResolver`, `SetEffectResolver`, `EnableJSEval`
- **`sobekRuntime`** -- concrete implementation using Sobek. Evaluates ES modules, calls the exported `main()` function, and returns `EvalResult` with result + trace.
- **Effect system** -- host functions (like `fetch`) are registered via `SetEffectResolver(name, fn)`. Each effect receives a `TraceRecord` and is responsible for recording its own input/output. Effects are available as global functions in JS. Registered effects are tracked so child runtimes (via `js.eval`) inherit them.
- **`js.eval`** -- nested JS evaluation. Enabled via `EnableJSEval()`. Exposes a global `js` object with `eval(source, args)` method. Creates a child `sobekRuntime` with the same effects and module resolver. Returns `{ result, error, traces }`.
- **Trace** -- `map[effectName]map[serializedInput][]serializedOutput`. Accumulated during `EvalToString`, reset on each call. Effects call `tr.SetInput()`/`tr.SetOutput()` to record.

### Module Loading (`anyruntime/anytype_loader.go`)

- `NewAnytypeLoader(cfg)` returns a `ModuleLoader` compatible with `SetModuleResolver`
- Resolves `"name@version"` modules by fetching Anytype program objects via REST API
- Extracts JS source via `ExtractMainSource` with fallback priority:
  1. Code block marked with `// __main_source` as first line
  2. First `` ```js `` or `` ```javascript `` tagged code block
  3. First code block of any kind (needed because Anytype API returns bare `` ``` `` without language tags)
- Program name property key can be either `__anytype_program` or `__anytype_program_name` (loader checks both)

### Host Functions (`runtime/hostfn/`)

- **`fetch`** -- synchronous HTTP fetch mimicking JS `fetch(url, opts)`. Supports `method`, `headers`, `body` options. Returns `{ok, status, statusText, headers, body}`. JSON response bodies (`Content-Type: application/json`) are auto-parsed into objects.
- **`fetchBatch`** -- concurrent HTTP fetch for multiple requests.
- **`sleep`** -- pause execution for N milliseconds.
- **`chatReply`** -- output a chat message to the configured writer.

## Build & Test

```
go build ./...
go test ./...
go run . <script.js>
```

---
> Source: [anyproto/anytype-agent-runtime](https://github.com/anyproto/anytype-agent-runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
