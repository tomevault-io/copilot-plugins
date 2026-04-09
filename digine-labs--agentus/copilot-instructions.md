## agentus

> Domain-specific language for AI agent orchestration. Compiles `.ags` source to bytecode IR, executed by a register-based VM. Implemented in Rust.

# Agentus

Domain-specific language for AI agent orchestration. Compiles `.ags` source to bytecode IR, executed by a register-based VM. Implemented in Rust.

## Project Goals

- Build a complete, correct DSL for AI agent orchestration
- Maintain a clean compilation pipeline: source -> lexer -> parser -> sema -> codegen -> runtime
- Keep the VM register-based, deterministic, and testable
- Provide a clean `HostInterface` boundary so the VM is decoupled from LLM providers

## Non-Goals / Constraints

- **Do NOT break bytecode/module format** unless a migration plan exists and is documented in `claude-progress.txt`
- Do not add runtime dependencies on specific LLM providers (all LLM interaction goes through `HostInterface`)
- Do not introduce async/tokio into the VM core (cooperative scheduling will use explicit yield points)
- Do not change instruction encoding width (32-bit fixed) without a documented rationale

## Session Boot Routine

Every new session MUST start with:
1. Read this file (`CLAUDE.md`)
2. Read `claude-progress.txt` for current status, what's broken, next steps
3. Read `spec/feature-checklist.md` for the definitive feature status
4. Run `make smoke` (or `cargo test --workspace`) to verify the repo is healthy
5. Then proceed with the task at hand

## Quick Reference

```bash
make build                    # cargo build --workspace
make test                     # cargo test --workspace
make fmt                      # cargo fmt --all
make lint                     # cargo clippy --workspace
make smoke                    # build + test + run 2 examples (fast verification)
make run-example EX=hello.ags # run a specific example

# Or directly:
cargo test --workspace
cargo run -p agentus-cli -- exec examples/hello.ags
```

## Architecture

8-crate Cargo workspace. Pipeline: **source -> lexer -> parser -> sema -> codegen -> runtime**

| Crate | Purpose | Key files |
|-------|---------|-----------|
| `agentus-common` | Shared types (Span, errors) | `src/lib.rs`, `src/span.rs` |
| `agentus-lexer` | Tokenizer with string interpolation state machine | `src/lexer.rs`, `src/token.rs` |
| `agentus-parser` | Recursive descent + Pratt parsing -> AST | `src/parser.rs`, `src/ast.rs` |
| `agentus-sema` | Name resolution, scope checking | `src/resolver.rs` |
| `agentus-ir` | Bytecode IR: opcodes, instructions, module format | `src/opcode.rs`, `src/instruction.rs`, `src/module.rs` |
| `agentus-codegen` | AST -> bytecode compiler | `src/compiler.rs` |
| `agentus-runtime` | Register-based VM, host interface, values | `src/vm.rs`, `src/host.rs`, `src/value.rs` |
| `agentus-cli` | CLI entry point (exec, compile commands) | `src/main.rs` |

See `docs/architecture.md` for the full architecture document.

## VM Design

- Register-based, 256 registers per call frame
- 32-bit fixed-width instructions (ABC, ABx, AsBx formats)
- 67 opcodes across 16 categories
- Three-tier memory: registers (local), agent memory (persistent), global (shared)
- `HostInterface` trait = boundary between VM and outside world (LLM, tools)

## Instruction Sequences

- **Regular function calls**: 2-instruction — `Call(result_reg, func_idx)` + `Nop(0, first_arg_reg, num_args)`
- **Tool calls**: 2-instruction — `TCall(result_reg, tool_desc_idx)` + `Nop(0, first_arg_reg, num_args)`
- **Iterator advance**: 2-instruction — `IterNext(var_reg, jump_offset)` + `Nop(0, iter_reg, 0)`
- **Method calls**: 3-instruction — `Call(result_reg, 0xFFFE)` + `Nop(0, first_arg_reg, num_args_with_handle)` + `Nop(0, method_name_const_idx)`

The sentinel value `0xFFFE` in the Call instruction's Bx field signals method dispatch rather than a regular function call.

## Key Patterns (Gotchas)

- **FunctionEmitter sub-emitters**: When compiling function/method bodies, create a new `FunctionEmitter` with `self.builder`. Extract `instructions` and `next_register` from it before touching `self.builder` again (borrow checker).
- **Multi-arg calls**: Compile ALL arguments first to get their registers, THEN copy to consecutive registers. Don't interleave compile+alloc.
- **Agent method frames**: `CallFrame.agent_id` is set when entering a method. `MLoad`/`MStore` use this to find the right `AgentInstance.memory`.
- **Module.agents/tools**: The `Module` struct has `agents: Vec<AgentDescriptor>` and `tools: Vec<ToolDescriptor>` fields. Any test that creates a `Module` manually must include `agents: Vec::new()` and `tools: Vec::new()`.
- **Sub-emitter table propagation**: When creating sub-emitters for function/method bodies, propagate `function_table`, `agent_table`, and `tool_table` so nested code can call tools, functions, and agents.
- **Tool default params**: Filled at compile time — compiler loads default const values for missing args before TCall.

## Testing

- **Integration tests**: `crates/agentus-codegen/tests/end_to_end.rs` (71 tests)
  - `run(source)` — compile + run, returns `Vec<String>` of emit outputs
  - `run_with_host(source, host)` — same but with a custom `HostInterface`
  - `run_values(source)` — returns raw `Vec<Value>`
  - `expect_compile_error(source, expected)` — asserts compilation fails
- **Unit tests**: Inline in each crate (48 total across all crates)
- **Agent/tool tests**: Use `EchoHost` which returns the user prompt as the LLM response and formats tool calls as strings
- **Total**: 119 tests (as of last verification)

## How to Add a Language Feature

Follow this pipeline in order. Each step must be complete before the next.

1. **Lexer** (`crates/agentus-lexer/src/`)
   - Add new tokens to `token.rs` (`TokenKind` enum)
   - Add lexing logic to `lexer.rs`
   - Add unit test in `lexer.rs`

2. **Parser** (`crates/agentus-parser/src/`)
   - Add AST node(s) to `ast.rs` (new `Stmt` or `Expr` variant, plus struct if needed)
   - Add parsing logic to `parser.rs`
   - Add unit test in `parser.rs`

3. **Sema** (`crates/agentus-sema/src/resolver.rs`)
   - Add resolution logic for the new AST node (scope tracking, name checking)
   - Add unit test

4. **IR** (`crates/agentus-ir/src/`)
   - If new opcodes are needed: add to `opcode.rs` (`OpCode` enum)
   - If new module structures are needed: add to `module.rs`
   - Keep instruction encoding stable (32-bit, ABC/ABx/AsBx)

5. **Codegen** (`crates/agentus-codegen/src/compiler.rs`)
   - Add compilation logic in `FunctionEmitter` methods
   - Handle the new AST node in `compile_stmt` or `compile_expr`
   - Add unit test in `compiler.rs`

6. **Runtime** (`crates/agentus-runtime/src/vm.rs`)
   - Add opcode handling in the VM's main `match` in `execute()`
   - Add unit test in `vm.rs`

7. **Integration test** (`crates/agentus-codegen/tests/end_to_end.rs`)
   - Add end-to-end test using `run()` or `run_with_host()`

8. **Example** (`examples/`)
   - Add or update a `.ags` example demonstrating the feature

9. **Update docs**
   - Update `spec/feature-checklist.md` to mark the feature done
   - Update `claude-progress.txt` with what was accomplished

10. **Verify**: `make smoke` must pass

## How to Debug a Failing Program

1. **Reproduce**: `cargo run -p agentus-cli -- exec <file.ags>` — note the exact error
2. **Isolate**: Create a minimal `.ags` reproduction case
3. **Check the pipeline stage**:
   - Lexer error? → Check token output (add debug prints in lexer)
   - Parser error? → Parse error message will reference token/position
   - Sema error? → "undefined variable" or scope errors
   - Codegen error? → Usually a `compile_*` method returning `Err`
   - Runtime error? → VM error during execution (bad opcode, type mismatch, etc.)
4. **Inspect IR**: Add `eprintln!` in codegen to dump instructions, or use `Module`'s debug output
5. **Write a regression test**: Add a `#[test]` in `end_to_end.rs` that reproduces the issue
6. **Fix and verify**: `make test`

## Development Workflow Rules

- **Always run tests before claiming done**: `make test` or `cargo test --workspace`
- **Never mark features done without verification**: Run the feature's specific test and an example
- **Prefer small PR-sized changes**: One feature or fix per commit/session
- **Update `claude-progress.txt` every session**: Record what you did, what passed/failed, next steps
- **Keep `spec/feature-checklist.md` current**: Mark items DONE only after verification

## Implementation Status

- **Phase 1** DONE: Vertical slice (let, emit, literals, basic ops)
- **Phase 2** DONE: Expressions & control flow (if/else, while, for, functions, string interpolation)
- **Phase 3** DONE: Agent core (agent defs, exec blocks, agent memory, method dispatch, HostInterface)
- **Phase 4** DONE: Tools (tool declarations, default params, TCall opcode, host-provided implementations)
- **Phase 5** PARTIAL: Multi-agent (Send/Recv done, Wait/Kill/scheduler TODO)
- **Phase 6** DONE: Collections (map literals, map ops, list push/index-set, len())
- **Phase 7** DONE: Error handling + resilience (try/catch/throw, assert, retry, parse_json/to_json)
- **Phase 8** TODO: Pipelines (pipeline/stage syntax, PipelineRun opcode)
- **Phase 9** TODO: Polish (binary serialization for .agc, better error messages, LSP)

## File Extensions

- `.ags` — source code
- `.agc` — compiled bytecode (serialization not yet implemented)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Digine-Labs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Digine-Labs)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
