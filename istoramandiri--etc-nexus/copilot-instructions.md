## etc-nexus

> This file contains context and instructions for AI agents working on this project.

# Claude Agent Instructions

This file contains context and instructions for AI agents working on this project.

## Evolving This Document

**This file (CLAUDE.md) should evolve over time.** When you encounter issues, gotchas, or discover important patterns:

1. **Document problems and solutions** - If you hit an error or unexpected behavior, add it to the "Lessons Learned" section below
2. **Update outdated information** - If instructions no longer work, fix them
3. **Add missing context** - If you needed information that wasn't here, add it for next time

This helps future sessions avoid repeating the same mistakes.

## Lessons Learned

### [RESOLVED] TTD and Fake PoW for Pre-Merge Tests (2026-01-27)

**Problem:** Consensus tests failed for two reasons:
1. TTD was always set (defaulted to max int) causing beacon sync mode
2. Tests with `SealEngine: "NoProof"` failed PoW verification

**Solution Applied:**
1. Modified `mapper.jq` to only set TTD when `HIVE_TERMINAL_TOTAL_DIFFICULTY` is explicitly provided
2. Added `HIVE_SKIP_POW` handling in `geth.sh` to enable `--fakepow` flag

### [RESOLVED] Unknown Flag: `--nocompaction` (2026-01-27)

**Problem:** `flag provided but not defined: -nocompaction`

**Solution:** Removed `--nocompaction` from `geth.sh` block import command.

### Core-geth Has `--fakepow` Flag

Core-geth supports `--fakepow` to skip PoW verification during block import. This is essential for running ethereum/tests consensus tests which use `SealEngine: "NoProof"`.

### Hive Environment Variables for core-geth

Key environment variables the client should handle:
- `HIVE_SKIP_POW` - Set when tests use NoProof seal engine (enable `--fakepow`)
- `HIVE_TERMINAL_TOTAL_DIFFICULTY` - Only set for post-merge tests
- `HIVE_CHAIN_ID`, `HIVE_NETWORK_ID` - Chain configuration
- `HIVE_FORK_*` - Fork block numbers

### Use consensus-etc Suite for ETC Testing

**Preferred approach:** Use the `consensus-etc` suite, which automatically filters to ETC-compatible forks and targets only clients with the `etc` role:
```bash
# Recommended: use consensus-etc suite — pass multiple clients as comma-separated
./hive --sim ethereum/consensus --sim.limit consensus-etc --client core-geth,besu-etc,nethermind-etc

# Single client
./hive --sim ethereum/consensus --sim.limit consensus-etc --client core-geth

# Filter by fork or test category
./hive --sim ethereum/consensus --sim.limit "consensus-etc/Berlin" --client nethermind-etc
./hive --sim ethereum/consensus --sim.limit "consensus-etc/.*bcValidBlockTest" --client besu-etc
```

**Multiple clients:** Hive accepts comma-separated `--client` values. It builds all images and runs the suite against each client in a single invocation. Always prefer this over running separate commands per client.

The legacy `--sim.limit` approach still works but requires manual fork filtering.

### Use --sim.parallelism for Faster Test Runs

**Always pass `--sim.parallelism 4` when running consensus tests.** The default parallelism of 1 makes large test runs extremely slow. Use 4 for a good balance of speed and resource usage:
```bash
# Recommended: run with parallelism 4
./hive --sim ethereum/consensus --sim.limit consensus-etc --client core-geth --sim.parallelism 4

# For machines with more resources, can go higher
./hive --sim ethereum/consensus --sim.limit consensus-etc --client core-geth --sim.parallelism 8
```

Without `--sim.parallelism`, tests run serially (~5s per test = hours for full suite).

### ETC Clients Only Support Pre-Merge Forks

ETC clients (core-geth, besu-etc, nethermind-etc) only support forks up through Berlin/Spiral. Post-merge tests (Paris, Shanghai, Cancun) will fail with misleading "unknown client type" errors. The `consensus-etc` suite handles this automatically.

| ETH Fork | ETC Equivalent | Supported |
|----------|----------------|-----------|
| Frontier — Berlin | Frontier — Magneto | Yes |
| London | Spiral (partial) | Partial (no EIP-1559) |
| Paris (Merge) and later | N/A | No |

## Documentation Pattern

**SITREP.md** is the **single source of truth** for current status:
- Current test progress and results
- What's working / broken
- Recent session updates
- **Operation Log** at the end: Reverse chronological log of completed work

**TODO.md** contains **only future work**:
- Planned next steps
- Known issues to address
- Future features/improvements
- NO current progress, NO resolved items

**README.md** is a brief overview:
- Links to SITREP.md for status
- Links to TODO.md for plans
- NO duplication of status or progress

**When work is completed:**
1. Remove from TODO.md
2. Add entry to Operation Log in SITREP.md (reverse chronological)
3. Skills update SITREP.md with progress; TODO.md is manually curated

## Available Skills

Project-specific skills are defined in `.claude/skills/`:

- `/pickup` - Resume work from previous session: load SITREP.md, TODO.md, summarize state
- `/handoff` - Prepare for context handoff: sync docs, update promptlog, commit (run when context is full)
- `/sync-docs [--commit]` - Synchronize SITREP.md (status) and TODO.md (future plans only)
- `/wrapup [message]` - Update prompt log and commit all changes (run at end of significant sessions)
- `/promptlog` - Generate PROMPTLOG.md from session history
- `/submodule-push [name]` - Push submodule changes to fork
- `/hive-run [simulator] [--client name]` - Build and run Hive integration tests
- `/hive-progress [--update]` - Check progress of running Hive tests and update SITREP.md
- `/hourly-monitor [--max-iterations N]` - Start automated hourly loop to monitor tests, update SITREP.md, and commit
- `/report <topic>` - Create structured reports with Discord-friendly summaries

**End of session:** Run `/wrapup` to commit changes, or `/handoff` if context is full and work will continue in a new session.

## GitHub Access

The `gh` CLI is configured with a fine-grained Personal Access Token scoped to specific repositories.

### Authorized Repositories

- `IstoraMandiri/etc-nexus` (this repo)
- `IstoraMandiri/hive` (Hive fork)
- `IstoraMandiri/core-geth` (core-geth fork)

### Token Permissions

- **Contents**: Read and write (push commits, create branches)
- **Metadata**: Read (required)

Use `/submodule-push` skill for pushing changes in submodules.

## Project Structure

```
etc-nexus/
├── hive/                  # Submodule: IstoraMandiri/hive (fork of ethereum/hive)
│   ├── clients/
│   │   ├── core-geth/     # ETC client (Go)
│   │   ├── besu-etc/      # ETC client (Java)
│   │   └── nethermind-etc/ # ETC client (C#)
│   └── simulators/ethereum/consensus/
│       ├── main.go        # Includes consensus-etc suite
│       └── etc_forks.go   # ETC fork env mappings
├── core-geth/             # Submodule: IstoraMandiri/core-geth
├── nethermind-etc-plugin/ # Submodule: IstoraMandiri/nethermind-etc-plugin
├── .devcontainer/         # Docker-in-Docker dev environment
├── .claude/
│   ├── skills/            # Project-specific Claude skills
│   ├── hooks/             # Hook scripts
│   └── settings.json
├── CLAUDE.md              # This file - Agent instructions
├── SITREP.md              # Single source of truth for current status
├── TODO.md                # Future work only
├── README.md              # Brief overview with links
├── PROMPTLOG.md           # Session prompt history
└── reports/               # Technical reports and analysis
```

## Key Context

- **Goal**: Test ECIP-1120 and ECIP-1121 implementations using Hive
- **Hive**: Ethereum's integration testing framework using Docker containers
- **ETC clients**: core-geth (Go), besu-etc (Java), nethermind-etc (C#)
- Hive orchestrates client containers and runs test simulators against them
- Client definitions added for all three ETC clients in our Hive fork
- `consensus-etc` suite implemented for streamlined ETC testing

## Hive Basics

- Client definitions live in `hive/clients/<name>/`
- Simulators live in `hive/simulators/<category>/<name>/`
- Clients are Docker containers configured via environment variables (HIVE_*)
- ETC clients have the `etc` role in their `hive.yaml`
- Use `/hive-run` skill to build and run tests
- **Always use `--sim.parallelism 4`** (or higher) for consensus test runs

## References

- [Hive Documentation](https://github.com/ethereum/hive/tree/master/docs)
- [ETC Community Call #45](https://cc.ethereumclassic.org/calls/45) - Background on Hive for ETC
- [ECIP-1120](https://ecips.ethereumclassic.org/ECIPs/ecip-1120)
- [ECIP-1121](https://ecips.ethereumclassic.org/ECIPs/ecip-1121)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IstoraMandiri)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IstoraMandiri)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
