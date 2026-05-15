## lockstep

> Lockstep is designed to be invokable from AI coding agents

# Using Lockstep from AI agents

Lockstep is designed to be invokable from AI coding agents
(Claude Code, Cursor, GitHub Copilot, etc.) to verify concurrency
properties of BEAM code. This file documents the AI-agent contract.

## What an agent can do with Lockstep

| Task | How |
|------|-----|
| Verify a hypothesized race exists | Generate a Lockstep test, run it, parse the result |
| Reproduce a flaky test deterministically | Wrap the test body in `ctest`, capture seed |
| Replay a saved counterexample | `mix lockstep.replay --trace <path>` |
| Minimize a counterexample | `mix lockstep.shrink --trace <path>` |
| Verify a fix doesn't regress | Re-run the same test with same seed |

## CLI commands (Bash-callable)

These are the primary entry points for AI agents. All run via the
`mix` command in the project root:

### `mix test <file>` — run a Lockstep test

```sh
mix test test/path/to/race_test.exs --seed 1
```

Exit code 0 = no bug found. Non-zero = test failed.

If a Lockstep test detected a bug, output includes:
- `iteration: N` — the iteration that triggered
- `seed: M` — top-level seed
- `strategy: :pct/:pos/...` — strategy used
- `trace path: traces/<name>.lockstep` — saved trace file

The trace file path is the key handoff: subsequent commands operate
on that file.

### `mix lockstep.replay --trace <path>` — print a saved trace

```sh
mix lockstep.replay --trace traces/foo-iter4-seed1.lockstep
```

Prints metadata + human-readable schedule with pid aliases (P0,
P1, ...) and step numbers. Includes `<-- FAILED HERE` marker.

### `mix lockstep.replay --trace <path> --run "Mod.fn" --file <test_file>`

Re-execute a specific test function under the recorded schedule.
Deterministic — same trace + same code = same outcome.

### `mix lockstep.shrink --trace <path>` — minimize a counterexample

```sh
mix lockstep.shrink --trace traces/foo-iter4-seed1.lockstep
```

Outputs a smaller trace file that reproduces the same bug.
Useful before filing an issue.

### `mix lockstep.dump_trace --trace <path>` — print trace as terms

For programmatic parsing.

## Generating a Lockstep test from a hypothesis

The simplest pattern an agent should produce when it wants to
verify a race hypothesis:

```elixir
defmodule Lockstep.Mimic.MyRaceTest do
  use Lockstep.Test

  ctest "verify hypothesis: <plain English description>",
        iterations: 1000, strategy: :pos, seed: 1 do
    # Build minimal mimic of the production code path.
    # Use Lockstep.* wrappers everywhere instead of OTP defaults.

    # 1. Setup state
    {:ok, pid} = Lockstep.GenServer.start_link(MyServer, init_state)

    # 2. Spawn concurrent actors
    parent = self()
    for i <- 1..3 do
      Lockstep.spawn(fn ->
        # ... code that exercises the hypothesized race ...
        Lockstep.send(parent, {:done, i})
      end)
    end

    # 3. Collect results
    for _ <- 1..3 do
      Lockstep.recv_first(fn {:done, _} -> true; _ -> false end)
    end

    # 4. Assert invariant
    if not invariant_holds?(...) do
      raise "invariant violated: ..."
    end
  end
end
```

Key rules for agent-generated tests:
1. **Use `Lockstep.*` wrappers** for every send, spawn,
   GenServer.call, ETS op, atomics op. Bare OTP calls won't be
   intercepted.
2. **Use `Lockstep.send/2` in the test body** (not `send/2`).
3. **Use `Lockstep.recv_first/1`** (not `receive`).
4. **The test function's body raises on invariant violation** — that's
   how Lockstep detects the bug.

## Strategy selection heuristic

| Bug shape | Strategy |
|-----------|----------|
| Tight read-modify-write on atomics/ETS | `:pos` |
| Multi-process message-ordering races | `:pct` (default) |
| Spin-loop / liveness scenarios | `:fair_pct` |
| Unknown shape, exploring | `:random` |

For deep races (>5 sync points between cause and effect), bump
`iterations` to 10_000+ and try multiple strategies.

## Parsing Lockstep output

A Lockstep failure exits with code 1 and prints an exception block
that's parseable. Key fields:

```
** (Lockstep.BugFound)
Lockstep found a concurrency bug on iteration <ITER>.
  seed:        <TOP_SEED>
  iter seed:   <ITER_SEED>
  strategy:    :<STRATEGY>
  trace path:  traces/<NAME>.lockstep

Reason:
  <REASON>

Schedule:
  step 1  <EVENT>
  ...
  step N  <EVENT>  <-- FAILED HERE
```

Regex for parsing (Python-flavoured):

```python
import re
m = re.search(r"iteration (\d+).*seed:\s+(\d+).*strategy:\s+:(\w+).*trace path:\s+(\S+)",
              output, re.DOTALL)
if m:
    iteration, seed, strategy, trace_path = m.groups()
```

## Common pitfalls when agents generate tests

1. **Forgetting to use Lockstep wrappers.** Agent writes
   `GenServer.start_link(...)` and the GenServer is unmanaged.
   Lockstep can't schedule it.
2. **Pre-constructed bugs.** Agent writes a test that always
   raises (no scheduling needed). The "bug found at iteration 1"
   is not informative — restructure to be a real race.
3. **Drain too short.** Many test bodies need `drain(100)+` ticks
   to let timers fire and processes settle.
4. **Bad invariant.** Agent's invariant should fail ONLY in the
   bug case, not during normal flow.

## Ground truth: what to trust

- **Lockstep's deterministic finding** (same seed → same iteration
  → same trace) means the bug is REAL given the test setup.
- **Lockstep's negative result** (10000 iterations, no bug found)
  means the test setup didn't surface a bug under the strategies
  used. It does NOT mean the code is bug-free.
- **`<-- FAILED HERE` marker** points to the exact event where
  the assertion fired.
- **Causal slice** (default on) shows ~5-20% of trace events
  causally related to the failure.

## Skill bundle

The Anthropic Skill format file is at `skill/lockstep/SKILL.md`.
Agents using the Anthropic Agent SDK can load this via
`Skill.load("lockstep")`.

## CI integration

Recommended pattern for CI:

```yaml
# .github/workflows/concurrency-tests.yml
- run: mix test --include lockstep --seed 1 --max-failures 1
```

The `--seed 1` ensures the same iteration counts hit the same
schedule across runs. `--max-failures 1` stops on the first
failure to keep the trace artifact small.

Save `traces/` as an artifact so failures are debuggable.

## Filing a bug found by an agent

When an AI agent finds a bug in third-party code via Lockstep, the
filable artifact is:

1. Lockstep test file
2. Trace file (`traces/<bug>.lockstep`)
3. Output of `mix lockstep.replay --trace <path>` (the human-readable
   schedule)
4. Suggested fix + the `mix lockstep.replay --run` command that
   verifies the fix

Reproducibility is the value-add — the bug isn't "we saw it once",
it's "seed N + strategy X + iteration M reproduces every time."

---
> Source: [b-erdem/lockstep](https://github.com/b-erdem/lockstep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-15 -->
