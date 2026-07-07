## miso-agent

> `miso-agent` is a **pure Go library** (`github.com/curtisnewbie/miso-agent`) for building LLM-powered ReAct agents. It is built on top of the [Eino](https://github.com/cloudwego/eino) graph orchestration framework and the author's own [miso](https://github.com/curtisnewbie/miso) microservice framework. There is no `main.go`; this project is consumed as a dependency.

# miso-agent AGENTS.md

## Project Overview

`miso-agent` is a **pure Go library** (`github.com/curtisnewbie/miso-agent`) for building LLM-powered ReAct agents. It is built on top of the [Eino](https://github.com/cloudwego/eino) graph orchestration framework and the author's own [miso](https://github.com/curtisnewbie/miso) microservice framework. There is no `main.go`; this project is consumed as a dependency.

---

## Build / Lint / Test Commands

```sh
# Build all packages (must pass with zero errors after any code change)
go build ./...

# Static analysis
go vet ./...

# Format code (ALWAYS run before committing)
go fmt ./...

# Run a single test by name (preferred approach to avoid timeout from integration tests)
go test -run TestFunctionName -count=1 -timeout 30s ./package/...

# Examples
go test -run TestAgent_shouldEvictToolResult -count=1 -timeout 30s ./agentloop/...
go test -run TestBuiltinTools_ReadFile -count=1 -timeout 30s ./agentloop/...
go test -run TestTokenizer_PruneMessagesToTokenLimit -count=1 -timeout 30s ./agentloop/...

# Run all unit tests in a specific package
go test -count=1 -timeout 60s ./agentloop/...

# WARNING: `go test ./...` may hang indefinitely — some tests in testapi/ require
# live external services (Redis, LLM APIs). Always use -timeout or run by package.
```

---

## Package Structure

| Package | Path | Purpose |
|---------|------|---------|
| `agentloop` | `agentloop/` | Core ReAct agent loop: tool calling, skills system, token management, prompt building, file-backed storage |
| `agents` | `agents/` | Pre-built specialized agents: RuleMatcher, MaterialExtract, ExecutiveSummaryWriter, MemorySummarizer, DeepResearchClarifier, plus model factory |
| `prebuilt` | `prebuilt/` | Higher-level ready-to-use agents: CsvFormatAgent, ClassificationAgent, FactCheck, AccuracyCheck, RelevanceCheck, CategoryAnalyze, ContextualRetrieval, plus shared Eval wrappers |
| `tools` | `tools/` | Reusable `agentloop.Tool` implementations: TavilySearch, DifyRetrieval — pass via `AgentConfig.Tools` |
| `graph` | `graph/` | Eino graph wrappers: compilation, invocation with trace callbacks, Mermaid diagram visualization, `GenericOps` config |
| `memory` | `memory/` | Conversational memory: short-term (Redis-backed conversation list) and long-term (Redis-backed summary with auto-compaction) |
| `agentapi` | `agentapi/` | External API integrations: Tavily Deep Research and Background Check wrappers |
| `testapi` | `testapi/` | Smoke tests: exercises graph compilation for all agents to verify they build successfully |

---

## Code Style Guidelines

### Import Organization

Imports are grouped into **3 blocks** separated by blank lines, in this order:

```go
import (
    // 1. Standard library
    "context"
    "fmt"
    "strings"
    "sync"

    // 2. Third-party dependencies
    "github.com/cloudwego/eino/compose"
    "github.com/cloudwego/eino/schema"
    "gopkg.in/yaml.v2"

    // 3. Internal packages + miso framework (treated as one group)
    "github.com/curtisnewbie/miso-agent/graph"
    "github.com/curtisnewbie/miso/errs"
    "github.com/curtisnewbie/miso/flow"
    "github.com/curtisnewbie/miso/util/strutil"
)
```

### Naming Conventions

| Category | Convention | Examples |
|----------|-----------|---------|
| Structs (exported) | `PascalCase`, noun-based | `Agent`, `AgentConfig`, `ToolRegistry`, `FileStore` |
| Structs (unexported) | `camelCase` | `ctxKey`, `agentLoopState`, `taskInput`, `openAiModelConfig` |
| Interfaces | `PascalCase`, behavior-named, no `I` prefix | `FileStore`, `Tool`, `SelfInvokeTool` |
| Constructors | `NewXxx(...)` | `NewAgent`, `NewToolRegistry`, `NewMemFileStore` |
| Functional options | `WithXxx(...)` returning `func(o *Config)` | `WithTemperature`, `WithMaxToken`, `WithSkills` |
| Methods | verb-first | `Execute`, `Build`, `Load`, `Register`, `AddTodo` |
| Exported variables | `PascalCase` | `DeepseekBaseURL`, `BasePrompt` |
| Unexported variables | `camelCase` | `agentCtxKey`, `finishToolName`, `modelMaxToken` |
| Constants | `PascalCase` (exported), `camelCase` (unexported) | `BasePrompt`, `maxToken32k` |
| Files | `snake_case` | `skill_loader.go`, `tools_builtin.go`, `tools_artifact.go` |
| Input/Output structs | `XxxInput` / `XxxOutput` | `RuleMatcherInput`, `MaterialExtractOutput` |
| Args structs for tools | `XxxArgs` | `ReadFileArgs`, `WriteFileArgs` |

### Error Handling

Use the `github.com/curtisnewbie/miso/errs` package. **Do not use `fmt.Errorf`** (except when using `%w` in stdlib-only contexts).

```go
// Create a new formatted error
return errs.NewErrf("file not found: %s", path)
return errs.NewErrf("task cannot be empty")

// Wrap an existing error with context (most common)
return nil, errs.Wrapf(err, "failed to load skills from %s", source)
return nil, errs.Wrapf(err, "failed to initialize tokenizer for model %s", config.Model)

// Simple wrap without message
return nil, errs.Wrap(err)
```

- Always return errors up the call stack; never log-and-swallow.
- Functions that may fail return `(value, error)` tuples.
- Tool execution functions return `(string, error)` — the string is the LLM-visible result.
- Never `panic` for recoverable errors.

### Logging and Context

Every function that touches the agent loop receives `flow.Rail` as its **first parameter**. `Rail` is the miso context carrier for logging, tracing, and span management.

```go
rail.Infof("loaded %d skills from %s", len(skills), source)
rail.Debugf("pruning messages to fit token limit %d", limit)
rail.Warnf("skill file %s has no frontmatter", path)
rail.Errorf("tool execution failed: %v", err)

// Timing
start := time.Now()
defer rail.TimeOp(start, "execute_tool")

// Child spans for parallel work
childRail := rail.NextSpan()
```

### Testing Conventions

- Test files: standard `*_test.go` placed alongside source files (same package, internal testing)
- Test function naming: `TestTypeName_MethodName` or `TestTypeName_MethodName_Scenario`
  - Examples: `TestAgent_shouldEvictToolResult`, `TestBuiltinTools_ReadFile_NotFound`
- Use **table-driven tests** as the primary pattern:
  ```go
  tests := []struct {
      name        string
      input       SomeType
      wantErr     bool
      description string
  }{
      {name: "valid case", ...},
      {name: "error case", wantErr: true, ...},
  }
  for _, tt := range tests {
      t.Run(tt.name, func(t *testing.T) { ... })
  }
  ```
- Mock types are defined within the test file itself.
- Helper functions are unexported (e.g., `generateLargeContent`, `containsString`).

### Core Architectural Patterns

**Eino Graph Composition** — all agents follow this pattern:
1. Define `Input`/`Output` structs and an `Ops` struct for configuration
2. Constructor `NewXxx(rail, chatModel, ops)` builds the Eino graph
3. Graph nodes: lambda nodes → chat model node → tool nodes → branch (loop/finish) → END
4. Graph is compiled once; `Execute(rail, input)` invokes it per call

**Functional Options** — configuration is passed via `func(o *configStruct)` closures:
```go
model, err := NewOpenAIChatModel("qwen3-max", apiKey,
    WithTemperature(0.3),
    WithMaxToken(32768),
    WithRetry(3),
)
```

**Mutex-Protected State** — all mutable managers use `sync.RWMutex`:
- `RLock/RUnlock` for reads, `Lock/Unlock` for writes.

**Parallel Execution** — use `async.AsyncPool` + `async.NewAwaitFutures` from miso for fan-out work; use `slutil.SplitSubSlices` for batching.

**Tool Creation** — six factory functions, choose by need:

| Function | When to use |
|----------|-------------|
| `NewToolFunc` | Basic tool, map-based args |
| `NewCtxAwareToolFunc` | Needs `AgentContext` (store/todos), map-based args |
| `NewTypedToolFunc[T]` | Typed struct args via JSON; manual `ParameterInfo` map |
| `NewTypedCtxAwareToolFunc[T]` | Typed struct args + `AgentContext`; manual `ParameterInfo` map |
| `NewAutoTypedCtxAwareToolFunc[T]` | Typed struct args + `AgentContext`; schema auto-deduced via reflection — no `ParameterInfo` map needed. Returns `(Tool, error)`. **T must be a struct**, not a primitive. Use `desc:""` tag for field descriptions. |
| `NewStrCtxAwareToolFunc` | Single required `string` param + `AgentContext`; no struct or schema boilerplate needed |

### Documentation Comments

- Every **exported** type, function, and method must have a Go doc comment beginning with its name.
- Comments are concise (typically one sentence); multi-line lists use `//   -` bullets.
- Cross-references use bracket notation: `// See [NewToolFunc], [StringParam]`.
- Include an `// Example:` code block in doc comments for non-obvious factory functions.
- Struct fields that represent configuration should be individually documented.

---

## General Agent Rules

1. **Always update this file** when adding important patterns, conventions, or discoveries that future agents should know.
2. **Never change to an alternative solution** after presenting a plan to the user without explicit confirmation.
3. **Always verify the project compiles** (`go build ./...`) after modifying any Go code.
4. **Always run `go fmt ./...`** before finishing any Go code changes.
5. **For numerical calculations**, always use a Python script — never calculate manually.
6. **Design and implementation documentation** goes in `agentdoc/` (not in this file). Reference it here briefly so agents can look it up.

---

## Supplementary Documentation (`agentdoc/`)

If an `agentdoc/` directory exists, it may contain design and implementation documents such as:
- `agentdoc/architect.md` — overall architecture decisions
- `agentdoc/design.md` — detailed design notes
- `agentdoc/faq.md` — common questions and known pitfalls

Check `agentdoc/` before starting a non-trivial task to avoid duplicating prior work.

---
> Source: [CurtisNewbie/miso-agent](https://github.com/CurtisNewbie/miso-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
