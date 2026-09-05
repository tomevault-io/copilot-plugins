## aoc2025erl

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and run

```bash
rebar3 escriptize          # compile and produce _build/default/bin/aoc2025
./_build/default/bin/aoc2025 bench              # benchmark all days
./_build/default/bin/aoc2025 bench 1            # benchmark day 1 only
./_build/default/bin/aoc2025 bench 1 2          # benchmark day 1 part 2 only
./_build/default/bin/aoc2025 bench --runs 50    # cap at 50 iterations per solution
./_build/default/bin/aoc2025 1                  # run day 1 (reads inputs/day01.txt)
./_build/default/bin/aoc2025 1 path/to/file     # run day 1 with explicit input file
```

There are no automated tests; correctness is verified by the benchmark harness comparing against `inputs/day*.answers`.

## Project structure

- `src/dayNN.erl` — solution modules, each exporting `solve_part1/1` and `solve_part2/1` taking a filename
- `src/aoc2025.erl` — escript entry point; dispatches `bench` subcommand or runs a single day
- `src/benchmark.erl` — benchmark harness: times each solution, checks answers, prints a Unicode table
- `src/aoc2025erl.app.src` — OTP app metadata
- `inputs/day*.txt` — puzzle inputs (git-ignored)
- `inputs/day*.answers` — two-line files: part 1 answer on line 1, part 2 on line 2

## Adding a new day

1. Create `src/dayNN.erl` exporting `solve_part1/1` and `solve_part2/1`.
2. Add two entries to the `solutions/0` list in `src/benchmark.erl`.
3. Place the input at `inputs/dayNN.txt` and optionally the expected answers at `inputs/dayNN.answers`.

## Benchmark harness behaviour

- Default runs: 10 (defined as `?DEFAULT_RUNS` in `benchmark.erl`).
- Auto-stops after 5 seconds total per solution regardless of run count.
- Answer column width is capped at 20 chars (truncated with `...` if longer).
- Green time = < 1 ms, red = ≥ 1 ms. Green ✓ = matches `.answers` file.

## Rust reference implementation

`~/src/github/aoc2025rust` contains solutions to the same problems in Rust and is used as a reference for algorithms and expected answers. When optimising Erlang solutions, check the Rust comments for O(1)/analytical approaches.

---
> Source: [jesperes/aoc2025erl](https://github.com/jesperes/aoc2025erl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
