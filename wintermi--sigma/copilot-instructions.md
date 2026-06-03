## sigma

> **Don't assume. Don't hide confusion. Surface tradeoffs.**

# Behavioural Guidelines

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## 5. Project Guidelines

**Respect the structure. Match conventions. Maintain contracts.**

### 1. Testing Standard

#### Write Behavioural Tests, Not Value Snapshots

Tests must verify **behaviour, logic, and edge cases** — not re-state what the source code already says.

**DO NOT write tests that:**
- Assert every field of a struct matches its hardcoded default (snapshot tests of constructors)
- Mirror the implementation line-by-line (if the test breaks only when the code changes intentionally, it has zero safety value)
- Test that standard library functions work (e.g. `os.Getenv` returns a value, `time.ParseDuration` parses `"30s"`)
- Test simple passthrough/assignment with no branching or transformation
- Assert log output contains specific field names (fragile, cosmetic)

**DO write tests that:**
- Cover branching logic and edge cases (invalid input, missing values, fallback behaviour)
- Test shared helper functions directly instead of indirectly through every call site
- Verify error paths and boundary conditions
- Test business rules, precedence, and override logic
- Smoke-test functions that could panic on unexpected input

**Ask yourself:** "If this test didn't exist, could a real bug slip through?" If the answer is no, don't write it.

#### Test Parallelism

Prefer `t.Parallel()` when a test is safe to run concurrently, but do not add it blindly — not every test can run in parallel.

```go
func TestSomething(t *testing.T) {
    t.Parallel()
    // ... test body ...
}
```

**When to add `t.Parallel()`:**
- Tests that only operate on locally constructed structs, local buffers, or `t.TempDir()` are good candidates
- Table-driven subtests where each case is independent can benefit from parallel execution:

```go
for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        t.Parallel()
        // ... use tt ...
    })
}
```

**When NOT to add `t.Parallel()`:**
- Tests that call `t.Setenv()` — Go panics at runtime if combined with `t.Parallel()`
- Tests that mutate package-level variables or shared global state
- Tests that depend on ordering or sequential side effects (e.g. shared files, ports, singletons)
- Tests where adding parallelism would require significant refactoring for unclear benefit

**Guidance:**
- Use your judgement — parallelism is a performance optimisation, not a correctness requirement
- When reviewing or writing tests, look for straightforward opportunities but don't force it

### 2. Development

> **Note:** All `mise run` commands must be executed from the project root directory.

**ALWAYS use these commands:**

```bash
mise run mise:validate   # Validate the local mise task configuration
mise run clean           # Clean the build directory
mise run go:build        # Build the CLI binary
mise run go:test         # Run the Go test suite
mise run go:race         # Run the Go test suite with the race detector
mise run go:vet          # Run go vet
mise run go:fmt          # Apply Go formatters defined in .golangci.yaml
mise run go:fmt:check    # Check Go formatting against .golangci.yaml
mise run go:lint         # Run golangci-lint using .golangci.yaml
mise run ci              # Run the local CI-equivalent verification suite
```

**NEVER run tools directly:**
- DO NOT RUN `go build`, `go test`, `go vet`, `golangci-lint`, or other tools directly
- Always use the `mise run` wrapper commands above
- This ensures consistent configuration and flags across all runs

### 3. Git Rules (CRITICAL)

**Stealth mode is ENABLED** - The agent must NOT perform git network operations.

- NEVER run `git push` or `git pull` - stealth mode only
- Only `git commit` is allowed

**CRITICAL RULES:**
- NEVER run `git push` or `git pull` - the human will handle remote sync unless requested directly
- Only local commits are allowed
- All commit messages must follow the Conventional Commits standard
- Commit messages must not contain any reference or co-authorship to the agent or model, including names such as Claude, Codex, or Gemini

---
> Source: [wintermi/sigma](https://github.com/wintermi/sigma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
