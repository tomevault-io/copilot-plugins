## lang-ex

> Graph-based agent orchestration for Elixir. Builds stateful, multi-step LLM workflows using nodes, edges, conditional routing, state reducers, human-in-the-loop interrupts, and checkpointing. Inspired by LangGraph, built on BEAM primitives.

# LangEx

Graph-based agent orchestration for Elixir. Builds stateful, multi-step LLM workflows using nodes, edges, conditional routing, state reducers, human-in-the-loop interrupts, and checkpointing. Inspired by LangGraph, built on BEAM primitives.

- **Version**: 0.5.0, **Elixir**: ~> 1.16
- **Deps**: `req`, `jason`, `telemetry`; optional `redix`, `postgrex`, `ecto_sql`
- **Test**: ExUnit with `mimic` for mocking

## Commands

```bash
mix deps.get                          # Install dependencies
mix compile --warnings-as-errors      # Compile (0 warnings required)
mix test                              # Run all tests
mix test path/to/test.exs:42          # Run specific test
mix format                            # Auto-format
mix format --check-formatted          # Check formatting
```

Always run `mix compile --warnings-as-errors` before considering work done. Zero warnings required.

## Architecture

```
LangEx (facade: invoke/3, stream/3)
├── Graph                             # Builder: new, add_node, add_edge, compile
│   ├── Graph.Compiled                # Compiled executable graph
│   ├── Graph.Pregel                  # Super-step execution engine (internal)
│   ├── Graph.State                   # State management with reducers
│   └── Graph.Stream                  # Lazy event streaming
├── LLM                               # Behaviour for provider adapters
│   ├── LLM.Anthropic                 # Claude adapter (streaming SSE)
│   │   ├── Anthropic.SSE             # SSE state machine (internal)
│   │   └── Anthropic.Formatter       # Message wire format (internal)
│   ├── LLM.OpenAI                    # GPT adapter
│   ├── LLM.Gemini                    # Gemini adapter
│   ├── LLM.Resilient                 # Retry wrapper with backoff
│   ├── LLM.ChatModel                 # Graph node helper for LLM calls
│   └── LLM.Registry                  # Provider resolution by model string
├── Tool                              # Tool/function definition struct
│   ├── Tool.Node                     # Graph node for parallel tool execution
│   └── Tool.Annotation               # Error recovery guidance for LLM
├── Message                           # Chat message types (Human, AI, System, Tool)
├── Checkpoint / Checkpointer         # Pause/resume with Redis or Postgres
├── Command / Send / Interrupt        # Graph control flow primitives
├── Config                            # Layered config resolution
├── ContextCompaction                 # Context window budget enforcement
└── Telemetry                         # Telemetry event definitions
```

### Behaviours

| Behaviour | Callbacks | Purpose |
|-----------|-----------|---------|
| `LangEx.LLM` | `chat/2`, `chat_with_usage/2` (optional) | LLM provider adapters |
| `LangEx.Checkpointer` | `save/2`, `load/1`, `list/2` | Checkpoint persistence backends |

### Key Design Decisions

- **No GenServers for domain state** -- graph state lives in function arguments and checkpoints, not processes
- **Pregel execution model** -- discrete super-steps with parallel node execution via `Task.Supervisor`
- **Process dictionary for interrupts** -- `Process.put(:lang_ex_resume, value)` enables the interrupt/resume mechanism
- **Reducers for state merging** -- each state key can have a custom reducer `(old, new) -> merged`

## Module Hierarchy

```
lib/lang_ex.ex                        → LangEx (facade)
lib/lang_ex/
├── command.ex                        → LangEx.Command
├── config.ex                        → LangEx.Config
├── context_compaction.ex            → LangEx.ContextCompaction
├── interrupt.ex                     → LangEx.Interrupt
├── send.ex                          → LangEx.Send
├── telemetry.ex                     → LangEx.Telemetry
├── checkpoint/
│   ├── checkpoint.ex                → LangEx.Checkpoint
│   ├── checkpointer.ex             → LangEx.Checkpointer (behaviour)
│   ├── postgres.ex                  → LangEx.Checkpointer.Postgres
│   └── redis.ex                     → LangEx.Checkpointer.Redis
├── graph/
│   ├── graph.ex                     → LangEx.Graph
│   ├── compiled_graph.ex            → LangEx.Graph.Compiled
│   ├── pregel.ex                    → LangEx.Graph.Pregel (@moduledoc false)
│   ├── state.ex                     → LangEx.Graph.State
│   └── stream.ex                    → LangEx.Graph.Stream
├── llm/
│   ├── llm.ex                       → LangEx.LLM (behaviour)
│   ├── anthropic.ex                 → LangEx.LLM.Anthropic
│   ├── anthropic/sse.ex             → LangEx.LLM.Anthropic.SSE (@moduledoc false)
│   ├── anthropic/formatter.ex       → LangEx.LLM.Anthropic.Formatter (@moduledoc false)
│   ├── openai.ex                    → LangEx.LLM.OpenAI
│   ├── gemini.ex                    → LangEx.LLM.Gemini
│   ├── resilient.ex                 → LangEx.LLM.Resilient
│   ├── chat_model.ex                → LangEx.LLM.ChatModel
│   └── chat_models.ex               → LangEx.LLM.Registry
├── message/
│   ├── message.ex                   → LangEx.Message (+ nested structs)
│   └── messages_state.ex            → LangEx.MessagesState
├── migration/
│   ├── migration.ex                 → LangEx.Migration
│   └── v1.ex                        → LangEx.Migration.V1 (@moduledoc false)
└── tool/
    ├── tool.ex                      → LangEx.Tool
    ├── node.ex                      → LangEx.Tool.Node
    └── annotation.ex                → LangEx.Tool.Annotation
```

## Code Style

Non-negotiable. Every change must follow these rules.

### Never Do

- `if` or `else` in function bodies
- `case`/`cond` when function heads with pattern matching work
- Nesting deeper than 1 level inside a function body
- Grouped aliases: `alias Foo.{Bar, Baz}`
- `alias Foo.Bar, as: Baz`
- Section divider comments: `# --- Section ---`
- `maybe_`, `do_`, `_if_`, `_or_` in function names
- Declaring a variable to use it exactly once
- `Enum.reduce` when `Enum.map |> Enum.sum` expresses intent better
- Missing `@spec` on public functions
- Missing `@moduledoc` on modules

### Always Do

- Pattern match in function heads for dispatch
- Guard clauses (`when`) for type/value checks
- Single-expression function bodies (one pipeline or `with`)
- Pipe operator for data transformation chains
- `with` for chaining fallible operations
- One alias per line, alphabetical
- Module names that mirror directory paths
- Test directory structure that mirrors lib
- Inline mock setup in every test via `Mimic.expect/3` or `Mimic.stub/3`
- Pattern-matching assertions: `assert %Message.AI{content: "hello"} = msg`

### Module Organization

Inside each module, order declarations as:

1. `@moduledoc`
2. `use` / `import` / `require`
3. `alias` (alphabetical)
4. Module attributes (`@constants`)
5. Types and struct
6. Public functions (with `@doc`, `@spec`)
7. Private functions

## Gotchas

- **Optional deps are compile-time guarded**: `postgres.ex`, `redis.ex`, and `v1.ex` are wrapped in `if Code.ensure_loaded?(Ecto)` / `if Code.ensure_loaded?(Redix)`. New optional-dep modules must follow the same pattern.
- **Process dictionary is used intentionally** in `Graph.Pregel` (interrupt/resume via `:lang_ex_resume`) and `LLM.Anthropic` (SSE streaming state). This is not a code smell here — it's the mechanism for cross-cutting concerns within a single execution.
- **`Mimic.copy` in `test/test_helper.exs`** must be updated when adding new modules that tests need to mock.
- **Ask before refactoring** beyond the immediate task. Style and structure changes require explicit approval.

## Workflow

### Adding New Modules

- Place the file where its module name dictates: `LangEx.Foo.Bar` -> `lib/lang_ex/foo/bar.ex`
- Add a corresponding test at `test/lang_ex/foo/bar_test.exs`
- If a directory gains 2+ related files, group them in a subdirectory
- Internal modules get `@moduledoc false`

### Adding New LLM Providers

1. Create `lib/lang_ex/llm/provider_name.ex` implementing `@behaviour LangEx.LLM`
2. Implement `chat/2` (required) and optionally `chat_with_usage/2`
3. Register in `LangEx.LLM.Registry` with prefix patterns
4. Add tests at `test/lang_ex/llm/provider_name_test.exs` using `Mimic.stub/3`

### Adding New Checkpointers

1. Create module implementing `@behaviour LangEx.Checkpointer`
2. Implement `save/2`, `load/1`, `list/2`
3. Wrap in `if Code.ensure_loaded?(Dep)` for optional dependencies

---
> Source: [surgeventures/lang_ex](https://github.com/surgeventures/lang_ex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
