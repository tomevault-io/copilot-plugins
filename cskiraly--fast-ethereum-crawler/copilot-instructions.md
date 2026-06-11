## fast-ethereum-crawler

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & run

```bash
make -j4 update    # one-time: init submodules and build the vendored Nim compiler
make -j4           # build build/dcrawl
./run.sh [args]    # creates results/<ISO-timestamp>/, runs the binary from there
./run.sh --help    # full CLI options
make clean         # removes build/dcrawl
```

`make` defers to `vendor/nimbus-build-system`. On a fresh checkout, the first `make` self-restarts after `git submodule update --init --recursive` populates the build system — this is intentional, not a hang. There is no test suite.

## Run-time layout

`run.sh` creates `results/<ISO-timestamp>/`, snapshots `git diff/status/describe` into it, then `cd`s in before launching `build/dcrawl`. All paths the binary writes (`peerstore.csv`, `discovery.csv`, the chronicles JSON log) are relative to that directory, so historical output lives under `results/*/`.

## Architecture

Single-file Nim program (`dcrawl.nim`, ~360 lines) built on Status Research's `eth/p2p/discoveryv5` (in `vendor/nim-eth`, currently the `cskiraly/nim-eth` fork — relevant when tracing protocol behavior).

Flow:
1. `DiscoveryConf` (confutils) parses CLI; defaults live as field annotations on the type.
2. `run()` sets up NAT, builds the `discv5_protocol.Protocol`, optionally starts a Prometheus HTTP server, then `waitFor discover(...)`.
3. `discover()` is the measurement loop. Maintains four `HashSet[Node]`s — `queuedNodes`, `measuredNodes`, `failedNodes`, `discoveredNodes`. Each iteration pops one node and fires `measureAwaitOne` without awaiting (concurrent), separated by `--query-interval-us` sleeps.
4. `measureOne()` issues `findNode` at distance `256 - (d mod 16)` (cycling through the 16 nearest buckets), records RTT/BW from `node.stats`, and appends one row to `peerstore.csv` for the queried node plus rows to `discovery.csv` for newly-seen ENRs.
5. When `queuedNodes` empties, the measured set is recycled as the next cycle's queue. Loops `--cycles` times.

Node identity in the `HashSet`s is **public-key based**, not ENR-seqnum based — re-discovering a node with a newer ENR keeps the older record. See the `TODO: check ENR versions` in `measureOne`.

`ethDataExtract()` pulls Ethereum-specific ENR fields (`secp256k1`, `eth2` fork digest, `attnets` bitfield, `client` per EIP-7636 — note: 7636 was withdrawn so this field is unreliable) and is the integration point for adding ENR-derived CSV columns.

### Optional libp2p identify probe

`--identify` enables a second-stage probe in `identify.nim`: a single `Switch` (TCP + noise + yamux + mplex) is built once at startup and reused for thousands of concurrent dials. After each successful `measureOne`, `identifyOne` derives the libp2p PeerId from the ENR's `secp256k1` pubkey, dials `/ip4/<ip>/tcp/<port>`, lets nim-libp2p auto-run identify, reads `AgentBook` / `ProtoVersionBook` / `ProtoBook` from `peerStore`, then disconnects. Output goes to `agents.csv`. Most peers are unreachable on TCP (firewalled / NATed), so coverage is partial — typical mainnet hit-rate is ~10-15% of measured peers, enough for a real client-type distribution. Tunables: `--identify-timeout-s`, `--identify-max-connections`, `--agents-file`. The libp2p import surface is restricted to specific submodules in `identify.nim` (not the umbrella `import libp2p`) because the umbrella pulls in QUIC, which isn't vendored.

## Logging & metrics

`dcrawl.nim.cfg` pins chronicles to two **streams** (per-sink filtering): the default stream (`defaultChroniclesStream`) writes textlines to stdout at `--log-level` (default INFO), and `progressLog` writes JSON to `dcrawl.log`. Per-peer progress events that would flood stdout — `findNode finished`, `discoveredNew`, `discoveredOld`, `identify ok`, `identify connect failed` — are emitted via `progressLog.info` and only land in the JSON file. `postprocessing/progress.py` (`read_json("dcrawl.log", lines=True)`) reads those messages by name, so the message names are a public interface — renaming them breaks the postprocessing.

Prometheus metrics are off by default; enable with `--metrics --metrics-port 8008`.

## Postprocessing

`make plots` from the repo root renders both scripts against the most recent `results/<timestamp>/` directory. Both scripts use PEP 723 inline metadata, so `uv run --no-project <script>` resolves dependencies on its own; both read from CWD, so when invoking manually you have to `cd` into the run dir first.

- `postprocessing/plot.py` — reads `peerstore.csv` + `discovery.csv` (plus `agents.csv` if present), writes `rtt-hist.png`, `rtt-cdf.png`, `bw-hist.png`, `bw-cdf.png`, `forkDigest.png`, `forkDigestAll.png`, and `clients.png`.
- `postprocessing/progress.py` — reads `dcrawl.log` (chronicles JSON sink), plots queue depth and discovery rate over time.

---
> Source: [cskiraly/fast-ethereum-crawler](https://github.com/cskiraly/fast-ethereum-crawler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
