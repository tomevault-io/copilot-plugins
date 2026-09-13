## agentic-optimizer

> This file provides guidance when working with code in this repository.

# CLAUDE.md

This file provides guidance when working with code in this repository.

## DO NOT EVER USE ALLOWS WITHOUT EXPLICIT USER PERMISSION. EVER.

This is the most important rule.

## DO NOT SIMPLIFY, RELAX, OR DELETE TESTS. EVER.
## Error-Focused testing is always the strategy.

Tests are meant to CATCH errors. If you delete them, simplify, relax, or ANY OTHER VARIATION of reducing test strictness,
you remove the point of having tests in the first place. You want tests to error, that means you found a bug. Instead,
analyze it, figure out the root cause, and fix it. You can modify the tests if they are incorrect, but do not simply
delete or simplify them to get the error to go away. This is your number one priority directive, always, no matter what.

## Project Overview

**Merlin** is an intelligent AI coding assistant with multi-model routing, automatic task decomposition, and comprehensive validation.

**THIS IS NOT A PRODUCTION OR PUBLIC PROJECT:**
- No backward compatibility requirements
- Remove code instead of deprecating
- Breaking changes are acceptable and encouraged
- Focus on cleanliness over compatibility

**Core Features:**
- Multi-tier model routing with automatic escalation
- **Recursive Step-Based Execution** (see EXECUTION_MODEL.md)
  - Agent returns `String | TaskList`
  - Full tool access at all times
  - Exit requirements validate step completion
  - Hard/soft error classification with retry logic
- Parallel task execution with conflict detection
- Multi-stage validation pipeline
- Terminal UI with real-time progress monitoring
- TypeScript runtime for agent code execution

**Your workflow**
- Assess necessary changes and current implementation from module docs
- Make the changes
- Update module docs and any other relevant documentation with your changes
- Add/verify testing covers those changes, preferrably with fixtures
- Run verify.sh to confirm
- Do not make new documentation files unless explicitly asked to

## Architecture

**Cargo workspace with these main crates:**

- `merlin-core` - Fundamental types, traits, error handling
- `merlin-context` - Context management, file indexing
- `merlin-languages` - Language backends (Rust via rust-analyzer)
- `merlin-providers` - External API providers (Groq, OpenRouter, Anthropic)
- `merlin-local` - Local model integration via Ollama
- `merlin-routing` - Task analysis, model tier selection, metrics
- `merlin-agent` - Agent execution, validation, orchestration
- `merlin-cli` - Command-line interface and TUI
- `merlin-tooling` - TypeScript runtime, file operations, bash execution
- `integration-tests` - Fixture-based integration testing

**Model Routing Flow:**
1. Analyze task complexity and intent
2. Select appropriate model tier
3. Execute with tool registry
4. Validate results
5. Escalate to higher tier on failure (up to 3 retries)

**Task Execution:**
- Agent decides: Return string OR decompose into TaskList
- Recursive decomposition: Steps can themselves return TaskLists
- Exit requirements: Each step validates completion
  - Callback validators: `file_exists`, `file_contains`, `command_succeeds`, etc.
  - Pattern matching: Regex validation
  - Named validators: Integration with ValidationPipeline
- Context specification per step
  - File patterns (glob)
  - Previous step results
  - Explicit content injection
- Retry logic:
  - Hard errors → escalate model tier
  - Soft errors → retry with feedback
  - Max 3 attempts per step
- Transactional file operations with rollback support
- Conflict detection prevents concurrent file modifications

## Repository Rules

### Strict Linting

**Extremely strict clippy configuration:**
- ALL clippy lints denied (all categories: pedantic, nursery, restriction, etc.)
- `missing_docs` denied - all public items must have doc comments
- No panic macros: `.unwrap()`, `.expect()`, `todo!()`, `unimplemented!()`, `unreachable!()`
- No `println!()`/`eprintln!()` - use `tracing` macros instead
- All code uses `Result<T, E>` with proper error handling

**Critical requirements:**
- **NEVER add `#[allow]` or `#[cfg_attr(test, allow(...))]` without EXPLICIT user permission**
  - If clippy complains, FIX THE CODE, do not silence the warning
  - This applies even for "trivial" lints like `min_ident_chars` or `excessive_nesting`
  - Refactor code to satisfy clippy's requirements
  - The ONLY exception is if the user explicitly says "add an allow for X"
- Do NOT use `cargo clean` - on ICE, delete only `target/{debug,release}/incremental/`, not entire profile
- Must pass `./scripts/verify.sh` with zero errors before completion

### Code Quality

**Rust Edition 2024:**
- Prefer RPITIT (Return Position Impl Trait In Trait) over `async-trait`
- Leverage gen blocks and async generators

**Documentation:**
All public items need doc comments with:
- Brief one-line summary
- Detailed explanation for complex items
- `# Errors` section for `Result` returns
- `# Examples` when helpful

### Repository Maintenance

**When modifying this project:**
1. Keep CLAUDE.md updated with behavioral changes and new patterns
2. Do NOT add implementation comments ("Phase 3 implementation", "Deleted function here", etc.)
3. Update CLAUDE.md if adding new repository rules or critical constraints
4. Focus CLAUDE.md on **what to do**, not **what was done**

### Crate Documentation

**Each crate has a README.md that must be kept up-to-date:**

Located in `crates/<crate-name>/README.md`:
- `merlin-core/README.md` - Core types, traits, error handling
- `merlin-context/README.md` - Context management, semantic search
- `merlin-languages/README.md` - Language backends
- `merlin-providers/README.md` - External LLM providers
- `merlin-local/README.md` - Local model integration
- `merlin-routing/README.md` - Task routing and analysis
- `merlin-agent/README.md` - Agent execution and validation
- `merlin-cli/README.md` - CLI and Terminal UI
- `merlin-tooling/README.md` - Tool system
- `integration-tests/README.md` - Testing framework

**When making changes:**
1. Update the relevant crate's README.md with:
   - Module or public API changes
   - Test coverage changes
   - Changes in features
2. Keep README sections focused and concise
3. Update testing status when adding/removing tests

## Testing Philosophy

**ALWAYS prefer JSON fixtures over manual tests.** Fixtures provide reproducible testing across all components with minimal code duplication.

**Fixture location:** `crates/integration-tests/tests/fixtures/`

Fixtures auto-discovered by `crates/integration-tests/tests/unified_tests.rs` and run with `UnifiedTestRunner`.

**Tests should not be modified to match incorrect behavior**. This is not a production system, it will have issues,
rely on tests to find them and fix them.

**Tests should NEVER duplicate behavior**. For example, the fixture runner shouldn't re-implement input handling,
it should run the CLI and pass inputs to it.

**Tests should NEVER be relaxed**. Verification is the most important thing. Deleting/simplifying tests just to make them pass is never a good idea.
You can change them if necessary, but never make them less strict and lose out on finding potential issues.

**ALWAYS follow the fail by default strategy**. Tests should never pass with unexpected conditions. For example, if you're testing
the thread system, you should not test if thread_count > 0. Instead, check that thread_count == expected_thread_count.
Failure in tests is ALWAYS better than a false pass, tests are designed to catch issues.

**Fixtures are designed to find all paths users can find**. If something is out of scope or not reachable by fixtures,
that likely means it can't be reached at all. Analyze any issues like that, determine if fixtures need to be expanded,
or if the code its targetting isn't used, and go ahead with the proper fix. Do NOT just say something is "Out of scope"
or "Unreachable" or "Should be unit tested instead".

**Routing-Based Fixture Matching**: Fixtures use routing-based matching (not string pattern matching) to test agent behavior:
- Matches on `context_type` (task_decomposition, step_execution, validation, etc.)
- Matches on `prompt_type` (design, debug, validation, planning)
- Matches on `retry_attempt` (0 for first attempt, 1+ for retries)
- Matches on `previous_result` (soft_error, hard_error)
- Tests routing decisions, not query string formats
- Decouples tests from implementation details
- If no `routing_match` specified, strategy matches any query (for simple fixtures)

Fixture coverage can be tested by adding the --fixture flag and --cov flag to the verify script

### Test Modification Guidelines

**CRITICAL: NEVER delete tests/fixtures that reveal bugs or issues:**
- **If a test fails, it has discovered a problem - FIX THE CODE, not the test**
- Failing tests are valuable - they show what's broken
- Deleting a failing test destroys evidence of a bug
- Examples:
  - ✓ GOOD: Test fails because `thread_count` returns 0 → Investigate and fix thread tracking
  - ✗ BAD: Delete test because `thread_count` verification doesn't work
  - ✓ GOOD: Test expects error but gets success → Fix code to properly handle error case
  - ✗ BAD: Remove error expectation because "error handling isn't implemented yet"

**When test expectations need updating:**
- If a test expects incorrect behavior, update it to expect the correct behavior
- Changing test expectations to match correct behavior is allowed and encouraged
- Only delete tests if they truly duplicate existing coverage or test non-existent features
- Examples:
  - ✓ GOOD: Update test expecting `readFile` to return `null` for missing files to expect an error instead
  - ✗ BAD: Delete the test entirely because it was testing incorrect behavior
  - ✓ GOOD: Delete a test for `createIsolatedWorkspace()` function that was never implemented
  - ✗ BAD: Delete a test for error handling just because the current implementation doesn't handle errors correctly

**If coverage doesn't increase after adding fixtures:**
- This indicates the code path ISN'T REACHABLE from user actions
- Investigate WHY the code isn't being executed
- Possible causes:
  - Dead code that should be removed
  - Feature not wired up correctly
  - Timing/async issue in test infrastructure
- DO NOT delete the fixture - use it to guide investigation

### When to Add Tests

**DO add a fixture if:**
- Testing new functionality not covered by existing fixtures
- Adding a new tool or capability
- Testing edge cases or error conditions

**DO NOT add a test if:**
- It duplicates existing fixture coverage
- Before adding, search: `rg "pattern" crates/integration-tests/tests/fixtures/`

**Unit tests** (inline `#[cfg(test)]`):
- Only for testing internal logic of private functions
- Must not duplicate fixture coverage

**Benchmarks** (`benchmarks/crates/`):
- Performance regression testing only

### TUI Testing

**NEVER manipulate TUI state directly.** Always use fixtures:
- `InputEventSource` trait enables fixture-based event injection
- Test TUI behavior through fixture events, not by calling TUI methods
- Verify UI state through fixture `verify.ui` blocks

## Running Verification

**Development (fast):**
```bash
./scripts/verify.sh
```

**Before commits (with coverage):**
```bash
./scripts/commit.sh
```

**Run specific tests with nextest:**
```bash
cargo nextest run -p <package> <test_name>
```

**Never use `cargo test` directly** - always use nextest or verify scripts.

## Critical Gotchas

### Error Handling

```rust
// BAD - will not compile
let value = result.unwrap();

// GOOD
let value = result.map_err(|err| RoutingError::ExecutionFailed(err.to_string()))?;
```

---
> Source: [BigBadE/agentic_optimizer](https://github.com/BigBadE/agentic_optimizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-13 -->
