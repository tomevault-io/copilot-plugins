## ncaa-rust2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

NCAA March Madness bracket optimizer written in Rust. Uses ELO ratings calculated from live game data (ESPN/NCAA APIs) or historical FiveThirtyEight CSV data to simulate tournament outcomes and optimize bracket picks using genetic algorithms and simulated annealing.

## Build and Run Commands

```bash
# Build the project
cargo build --release

# Run with default settings (ESPN data source)
cargo run --release

# Run with specific data source
cargo run --release -- --source espn    # ESPN API (default)
cargo run --release -- --source ncaa    # NCAA API
cargo run --release -- --source csv     # FiveThirtyEight CSV

# Generate portfolio of diverse brackets
cargo run --release -- --portfolio 5 --portfolio-strategy champion
cargo run --release -- --portfolio 5 --portfolio-strategy diverse
cargo run --release -- --portfolio 5 --portfolio-strategy annealing

# ELO-only mode (skip bracket optimization)
cargo run --release -- --elo-only

# Lock specific teams to reach rounds
cargo run --release -- --lock-team "Duke:FinalFour" --lock-team "UConn:Winner"

# Run tests
cargo test

# Run single test
cargo test test_expected_score
```

## Architecture

### Core Data Flow
1. **Data Ingestion** (`api.rs`, `ingest.rs`): Fetch game results from ESPN/NCAA APIs or CSV, parse into `GameResult` structs
2. **ELO Calculation** (`elo.rs`): Process games chronologically to calculate team ratings with margin-of-victory adjustments
3. **Tournament Setup** (`ingest.rs`): Create `TournamentInfo` with 64 teams organized by region and seed
4. **Bracket Simulation** (`bracket.rs`): Generate brackets using Monte Carlo simulation with probability-weighted outcomes
5. **Optimization** (`pool.rs`, `anneal.rs`): Evolve brackets using genetic algorithms or simulated annealing to maximize expected value
6. **Portfolio Generation** (`portfolio.rs`): Create diverse bracket portfolios with different champion strategies

### Key Structs

- **`TournamentInfo`**: Holds 64 teams with `RcTeam` (Arc<Team>) for efficient sharing, includes O(1) lookup map by (region, seed)
- **`Bracket`**: 63 games stored as flat vector (R1: 0-31, R2: 32-47, R3: 48-55, R4: 56-59, R5: 60-61, R6: 62), uses binary representation for mutations
- **`Game`**: Single matchup with win probabilities calculated via logistic function: `1 / (1 + 10^(-rating_diff * 30.464 / 400))`
- **`ScoringConfig`**: Configurable round scoring with seed bonuses (Add, Multiply, or None per round)

### Bracket Binary Encoding
Each bracket is represented as 63 booleans (`hilo`), where `true` means the lower seed (or alphabetically earlier region for cross-region games) advances. This enables efficient mutation and crossover operations.

### Parallel Processing
Uses `rayon` for parallel bracket generation and scoring. Teams use `Arc<Team>` for thread-safe reference counting without data cloning.

## CLI Arguments Reference

| Flag | Default | Description |
|------|---------|-------------|
| `--source` | espn | Data source: espn, ncaa, csv |
| `--season` | auto | Season format: "2024-2025" |
| `--tournament-year` | - | Year to fetch bracket teams |
| `--bracket-file` | - | Local JSON file with bracket teams |
| `--generations` | 200 | Genetic algorithm generations |
| `--batch-size` | 1000 | Monte Carlo simulations per scoring (legacy mode) |
| `--portfolio` | - | Number of brackets to generate |
| `--portfolio-strategy` | ga | Strategy: champion, diverse, annealing, ga |
| `--diversity-weight` | 5.0 | Weight for bracket diversity (legacy strategies) |
| `--lock-team` | - | Lock team to round (repeatable) |
| `--score-r1` through `--score-r6` | 1,2,4,8,16,32 | Points per round |
| `--seed-r1` through `--seed-r6` | add/multiply | Seed scoring mode per round |
| `--config` | - | Path to YAML configuration file |
| `--generate-config` | - | Print sample config.yaml and exit |
| `--optimization-mode` | ga | Single bracket mode: legacy, ga, hybrid |
| `--population-size` | - | Override GA population size |
| `--pool-size` | - | Override simulation pool size |
| `--smart-mutation` | - | Override smart mutation setting |
| `-v, --verbose` | false | Verbose output during optimization |

## Configuration File

Settings can be configured via `config.yaml` (auto-loaded if present):

```yaml
scoring:
  round_scores: [1.0, 2.0, 4.0, 8.0, 16.0, 32.0]
  seed_scoring: ["add", "add", "add", "multiply", "multiply", "multiply"]

ga:
  population_size: 100
  generations: 200
  mutation_rate: 0.15
  crossover_rate: 0.7
  elitism_count: 5
  tournament_size: 3
  smart_mutation: true
  smart_mutation_rate: 0.8

simulation:
  pool_size: 10000

portfolio:
  num_brackets: 5
  best_ball: true
```

Generate a sample config: `cargo run -- --generate-config > config.yaml`

## Genetic Algorithm (New)

The GA implementation (`src/ga.rs`) provides:

- **SimulationPool**: Pre-generates N random brackets for consistent scoring. Avoids regenerating random brackets each evaluation.
- **Best-Ball Scoring**: Portfolio fitness = average of max(scores) across simulations. Only one bracket needs to score well per simulation.
- **Smart Mutation**: Instead of random bit flips, picks a team and round, forces that team to advance to that round. Changes propagate to earlier rounds.
- **Sequential Portfolio Optimization**: Optimizes bracket 1, freezes it. Optimizes bracket 2 for marginal contribution to portfolio, freezes. Repeat.

Key commands:
```bash
# Single bracket with GA (default)
cargo run --release

# Portfolio with GA and best-ball scoring
cargo run --release -- --portfolio 5 --portfolio-strategy ga -v

# Hybrid SA+GA for single bracket
cargo run --release -- --optimization-mode hybrid -v

# Use legacy hill-climbing
cargo run --release -- --optimization-mode legacy
```

## Data Caching

API responses are cached in `./data/` directory:
- `games_{season}.json`: Season game results (6-hour staleness)
- `bracket_{year}.json`: Tournament bracket teams

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/corykiser) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
