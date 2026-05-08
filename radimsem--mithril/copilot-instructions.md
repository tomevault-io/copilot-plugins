## mithril

> Sandboxed MCP server that replaces the generic shell tool with validated, purpose-built execution tools for AI agents. No shell, no escapes — every command goes through typed arguments, injection guards, path confinement, and OS-level isolation.

# Mithril

Sandboxed MCP server that replaces the generic shell tool with validated, purpose-built execution tools for AI agents. No shell, no escapes — every command goes through typed arguments, injection guards, path confinement, and OS-level isolation.

## Quick Reference

```bash
cargo build                        # Build all crates
cargo build -p mithril             # Binary only
cargo test                         # All non-ignored tests
cargo test -- --ignored            # Include sandbox + ecosystem tests
cargo test -p server               # Single crate
cargo test -p server cat_reads_file # Single test by name
cargo clippy --workspace           # Lint
cargo fmt --all -- --check         # Format check
```

## Workspace Structure

```
crates/
  bin/        CLI binary entry point (clap args, config loading, MCP transport)
  common/     Shared types: Config, ExecContext, ToolDef/ToolExec traits, errors, validation rules
  patterns/   Compiled regex patterns for validation (melody DSL, lazy_static)
  tools/      ~160 tool implementations across 10 domains (system, git, docker, go, js, python, rust, gh, glab, run_chain)
  server/     MCP ServerHandler, tool/prompt/resource routing, execution pipeline, history buffer
  runner/     Process spawning layer: timeouts, stdout streaming, progress notifications
  validator/  Argument validation layer: injection guard, path confinement, env blocklist, elicitation
  sandbox/    OS isolation layer: BubbleWrap (Linux) / sandbox-exec (macOS) / Noop fallback
  proxy/      Optional RTK (Rust Token Killer) prefix layer for token-optimized output
```

## Architecture

### Execution Pipeline

Every tool call flows through 4 sequential layers:

```
MCP Request → Router → build_context() → [ Validate → Proxy → Sandbox → Runner ] → format() → MCP Response
```

1. **ValidateLayer** — checks argument types, string patterns, injection characters, path confinement, env blocklist, elicitation approval
2. **ProxyLayer** — optionally prepends `rtk` binary for supported commands
3. **SandboxLayer** — prepends OS-specific sandbox command prefix (bwrap/sandbox-exec)
4. **RunnerLayer** — spawns process via `tokio::process::Command`, streams stdout, enforces timeout

### Key Types

| Type | Location | Purpose |
|------|----------|---------|
| `ToolDef` | `common/src/tool.rs` | Trait: name, description, parameters, available, elicitable |
| `ToolExec` | `common/src/tool.rs` | Trait: Args type, build_context(), format() |
| `ExecContext` | `common/src/context.rs` | Full execution state passed through pipeline |
| `MithrilLayer` | `common/src/lib.rs` | Pipeline layer trait: `async fn apply(ctx) -> Result<ctx>` |
| `ValidationRule` | `common/src/validation.rs` | Enum: String, Integer, StrVector, PathConfined, AllowList |
| `SandboxLevel` | `common/src/config.rs` | Enum: Dir (default), Readonly, Container |
| `SandboxMeta` | `common/src/context.rs` | Struct: needs_network, ro_binds, rw_binds |
| `GlobalArgs` | `tools/src/lib.rs` | Struct: cwd, sandbox, envs, isolated_env (merged into every tool call) |
| `ToolRegistry` | `server/src/registry.rs` | Registers all tools, stores metadata + context builder closures |
| `ServerRouter` | `server/src/router.rs` | MCP routing: tool dispatch, prep_ctx, run_chain |
| `MithrilServer` | `server/src/lib.rs` | MCP ServerHandler: initialize, list_tools, call_tool, prompts, resources |

### Tool Implementation Pattern

Each tool is a struct implementing `ToolDef` + `ToolExec`:

- **Args struct** with `Deserialize + Serialize + JsonSchema` — defines the JSON schema
- **`build_context()`** — pushes CLI args to `ctx.cmd`, sets validation rules on `ctx.tool.validation_rules`, configures `ctx.sandbox.meta`
- **`format()`** — converts `ExecOutput` to MCP `CallToolResult`
- Tool files live in `crates/tools/src/<domain>/<tool_snake>.rs`
- Registration in `crates/server/src/registry.rs` via `register::<ToolStruct>()`
- Router method in `crates/server/src/router.rs` with `#[tool(name, description)]` macro

### Tool Domains

| Domain | Count | Binary | Examples |
|--------|-------|--------|----------|
| system | ~29 | Various | Cat, Ls, Grep, Rm, Sed, Find, Wc |
| git | 18 | git | GitStatus, GitCommit, GitDiff, GitLog |
| docker | ~27 | docker | DockerRun, DockerBuild, DockerExec |
| go | 12 | go | GoBuild, GoTest, GoModTidy |
| js | 13 | node/npm | Node, NpmInstall, Jest, Prettier |
| python | 17 | python/pip/uv | Python, Pytest, Ruff, UvRun |
| rust | 13 | cargo | CargoBuild, CargoTest, CargoClippy |
| gh | 11 | gh | GhPrList, GhIssueView, GhRunList |
| glab | 11 | glab | GlabMrList, GlabPipelineView |
| chain | 1 | — | RunChain (pipe/and composition) |

## Configuration

Config file: `.mithril.toml` (TOML, loaded at startup, defaults if missing)

```toml
[server]
timeout_secs = 180        # Process timeout
max_output_lines = 2048   # Output truncation
history_size = 8           # Tool call history buffer

[project]
ecosystem = "rust"         # Language/framework
root.abs_path = "/path"    # Project working directory

[sandbox]
default_level = "dir"      # dir | readonly | container
path_blocklist = [".env", ".env.*", "*.pem", "*.key", "*.p12", "*.pfx"]
```

CLI overrides: `--timeout`, `--max-output-lines`, `--history-size`, `--project-root`, `--ecosystem`, `--sandbox`

## Security Model

- **No shell** — commands are `Vec<OsString>`, never passed through a shell
- **Injection guard** — blocks `;|&<>$(){}` backticks, quotes, `=*?[]!` in arguments (per-rule exemptable)
- **Path confinement** — canonicalized paths must start with cwd, .git blocked by default
- **Path blocklist** — rejects .env, *.pem, *.key, *.p12, *.pfx
- **Env blocklist** — blocks LD_PRELOAD, LD_LIBRARY_PATH, PYTHONPATH, DYLD_*, etc.
- **Elicitation** — destructive tools (docker, etc.) require user approval via MCP elicitation protocol
- **Sandbox** — OS-level isolation wraps every command execution

## Conventions

Detailed rules are in `.claude/rules/`:
- `rust-patterns.md` — Rust code conventions (error handling, iterators, borrows, etc.)
- `cli-testing.md` — Test organization, harness usage, patterns and anti-patterns

Key points:
- `expect("reason")` in tests, `.context()`/`?` in production — never `unwrap()`
- `.to_owned()` over `.clone()` for string/value copies
- Iterator chains over loops (unless early `?` return needed)
- All structs derive `Debug`
- Tool tests go in `crates/server/tests/tools/` — tools crate has zero unit tests
- `TempEnv` for test directories (uses `/var/tmp/mithril_tests` to survive bwrap overlay)
- Availability guards for external binaries (early return, not `#[ignore]`)

---
> Source: [radimsem/mithril](https://github.com/radimsem/mithril) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
