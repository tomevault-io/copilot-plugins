## cacagpu

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

The search runs on an **NVIDIA GPU** via a CUDA kernel linked into the Go binary
with cgo, so the build is driven by `make` (nvcc + cgo), not plain `go build`.

```bash
# Build (nvcc compiles the kernel -> static lib -> cgo links the Go binary)
make                       # produces ./bitcoin_finder (a Linux ELF binary)
make CUDA_ARCH=sm_75       # override GPU arch (default sm_86 = RTX 3050)

# Run (from project root; reads data/ relative to the cwd)
./bitcoin_finder

# Tests (must build the GPU lib first; `make test` does both)
make test
CGO_ENABLED=1 go test -count=1 -run TestName .   # single test, uncached

# Clean build artifacts
make clean
```

Building requires `nvcc` (CUDA toolkit), a native Linux Go toolchain, and gcc.
`CGO_ENABLED=1` is required. `go test`'s result cache does **not** track the cgo
static library, so after recompiling the kernel use `-count=1` to force re-runs.

The program must be run from the project root directory, as it opens data files
relative to the working directory (`data/hash160s.json`, `data/ranges.json`,
optionally `data/wallets.json`).

**Build environment note (WSL):** the GPU is reachable from WSL through the WSL
CUDA driver, so build and run the Linux binary inside WSL. The Windows `go.exe`
(under `/mnt/c/Program Files/Go`) fails on the Linux filesystem with `RLock
go.mod: Incorrect function`; use a native Linux Go (e.g. `mise install go@1.22`).
Cross-compiling the CUDA+cgo binary to a Windows `.exe` from WSL is impractical,
so the committed `bitcoin_finder.exe` (the old CPU-only Windows build) is stale.

## Purpose

This is a CLI tool for participating in the **Bitcoin Puzzle challenges** (the well-known 1000 BTC / 32 BTC puzzle transactions, where keys for puzzles 1–160 are deliberately constrained to known ranges and solving them is the intended goal). The 160 wallets and their key ranges in `data/` correspond to these puzzle entries.

**This code exists solely for the puzzle challenge. It must never be adapted or used to attack regular wallets that don't belong to the puzzle.**

## Architecture

The tool searches private keys within a puzzle's defined range to find the key matching the target puzzle wallet's Hash160 (RIPEMD-160 of SHA-256 of the compressed public key).

### Flow

1. `main.go` — entry point: loads data, prompts for wallet number (1–160), resolves the target Hash160 and key range, then calls `searchForPrivateKey`.
2. `search.go` — GPU search driver. Generates the random per-walk seeds, drives the CUDA kernel `roundsPerLaunch` rounds at a time, logs throughput every 10 seconds (both the interval rate and the cumulative average), and on match writes the private key to `found_key_<hash160prefix>.txt`. See **Search algorithm** below.
3. `gpu.go` — cgo bindings to the CUDA library (`gpu/secp256k1_gpu.{h,cu}`): device detection, the search lifecycle (`gpuSearchInit`/`run`/`free`), and `gpuHash160Batch` (used only by the correctness test).
4. `gpu/secp256k1_gpu.cu` — the CUDA kernel: 256-bit field arithmetic mod p (64-bit limbs + `__int128`), Jacobian EC point add/double + base-point scalar mult, SHA-256 and RIPEMD-160, plus the seed/step/hash kernels. `gpu/secp256k1_gpu.h` is the `extern "C"` interface.
5. `bitcoin.go` — CPU **reference** primitives used to validate the GPU: `privateKeyToHash160` (`btcsuite/btcd` secp256k1 + `btcutil.Hash160`) and `privateKeyToAddress` (unused).
6. `data.go` — data loading: reads `data/hash160s.json` as the primary source; falls back to `data/wallets.json` (address strings) but conversion is not implemented.
7. `models.go` — JSON structs (`WalletData`, `RangeData`, `Range`, `Hash160Data`).
8. `colors.go` — ANSI terminal color constants.

### Data files (`data/`)

- `hash160s.json` — array of hex-encoded Hash160 values, one per wallet (preferred input)
- `wallets.json` — array of Bitcoin mainnet addresses (used only if `hash160s.json` is absent; conversion path is a stub)
- `ranges.json` — array of `{min, max, status}` objects with 0x-prefixed hex bounds; index aligns 1-to-1 with `hash160s.json`

### `temp/hash160_generator.go`

A standalone utility (its own `main` package) that converts `data/wallets.json` → `data/hash160s.json`. Run it from the `temp/` directory; it resolves paths relative to its parent directory.

### Search algorithm (GPU)

The whole hot loop runs on the GPU (~650 Mkeys/s on an RTX 3050). The design is
the VanitySearch-style "group + symmetry" scheme:

- **Walks.** `searchForPrivateKey` (`search.go`) starts `targetWalks` (64K, rounded
  to `gpuSearchWalkMultiple()`) independent walks — one per kernel thread. Each
  gets a uniformly-random start in `[minKey, maxKey]`. Random starts make each run
  sample different regions of the range.
- **Groups.** Each round, a walk covers the `GRP_KEYS = 2*GRP_HALF+1` keys *centred*
  on its current point: the centre itself plus `P ± iG` for `i = 1..GRP_HALF`. It
  then jumps the centre forward by `GRP_KEYS`, so successive rounds tile the key
  space contiguously — no gaps, no overlap.
- **One inversion per round.** The whole round costs a single `fe_inv`, for two
  compounding reasons:
  - Montgomery's trick batches the `GRP_HALF+1` denominators `(Px - x(iG))` into
    one inversion, amortizing its ~270 multiplies across the group.
  - The **± symmetry** halves how many denominators exist at all: `-Q` has the same
    x as `Q`, so `P+iG` and `P-iG` *share* the denominator `Px - x(iG)`. One
    inverse yields two keys.

  Together these take the cost from ~14.4 field multiplies per key (the old
  one-key-per-step design) to ~5.0.
- **Group table.** `k_gtab` precomputes `(i+1)*G` affine for `i < GRP_HALF` plus the
  jump point `GRP_KEYS*G`, once at startup. Every thread reads the same entry at the
  same time, so these loads broadcast out of cache.
- **Storage.** Only the batch *prefix* products are kept per thread; the denominators
  are recomputed on the backward pass from the (round-invariant) centre and the
  table, which is cheaper than keeping a second array of that size live.
- **Host loop.** `gpu_search_run` advances every walk by `roundsPerLaunch` rounds per
  synchronous launch; the centres live in device memory across launches so Go can
  report progress and check for a match between batches.
- **Match reporting.** The kernel records `(walk, signed offset)` and the host rebuilds
  the scalar from its seed copy — no 256-bit arithmetic in the hot loop.

#### Hashing

SHA-256 and RIPEMD-160 are specialized for their fixed-shape inputs, which matters
much more than it looks:

- SHA-256 hashes exactly one block (33-byte compressed pubkey), so the message tail
  is constant and `w[9..15]` are literals. The schedule is a **rolling 16-word
  window** — the earlier materialized `w[64]` could not fit the register budget and
  spilled hard.
- RIPEMD-160's schedule/shift tables (`RL/RR/SL/SR`) are deliberately **function-local
  `const`, not `__constant__`**. They index `X` and supply shift amounts, so they must
  constant-fold to literals; under full unrolling they do, keeping `X[16]` in registers.
  As `__constant__` arrays they stayed runtime values, forcing `X` into local memory
  plus a constant load on each of the 160 round-halves.
- Both hashes work in 32-bit words end to end; no byte arrays, and the target compare
  is five word compares.

#### Tuning notes

The kernel is **compute-bound** (64-bit modular multiplies), and the card is
**power-bound** at its 130 W cap (`power.max_limit` = 130 W, not raisable). Sustained
is ~650 Mkeys/s on puzzle #60; short runs read high because the card boosts above its
sustained clock for the first minute. **Benchmark with `./bench.sh`, and compare the
per-interval rate, not the cumulative average.** To measure anything <5% reliably,
lock clocks first: `sudo nvidia-smi -lgc 1800` / `-rgc`.

Measured ablations (`-DBENCH_NO_HASH` skips hashing but keeps the EC work):

- The hash rewrite alone took the kernel from ~174 to ~300 Mkeys/s. **Hashing was
  ~46% of runtime**, not "essentially free" — an earlier profiling note claimed the
  latter and was wrong. With the specialized hashes it is now nearly free again.
- The group + ± symmetry restructure took ~310 to ~650 Mkeys/s.
- `GRP_HALF` swept 64/128/256/512: 256 is best (~608 vs ~578 at 64), matching the
  ~5.0 vs ~6.6 multiplies-per-key prediction. Past 256 the prefix array's local
  memory outweighs the shrinking inversion share.
- `MIN_BLOCKS_PER_SM` 4/6/8 (= 126/80/64 registers) all land within noise; 6 is
  committed. Because the card is power-limited, small occupancy changes do not move
  the needle — do not chase them without locked clocks.

**Confirmed dead ends (do not retry without locked clocks):** hand-written PTX
carry-chain `mul_wide` (ptxas already schedules the `__int128` version better); a
dedicated Comba squaring (10 vs 16 limb-muls, but its data-dependent carry
propagation measured *slower*, so `fe_sqr` just calls `fe_mul`); warp/block-batched
inversion (a SIMT trap — one inversion per warp idles 31 lanes).

#### Correctness

Silent math bugs produce wrong hashes, not crashes, so run `make test` (with
`-count=1`) after any change to the kernel's field/curve/hash math. `gpu_test.go`:

- `TestGPUHash160MatchesReference` asserts the GPU field/curve/hash pipeline yields
  byte-identical hash160s to the CPU reference `privateKeyToHash160` across fixed +
  random keys.
- `TestGPUSearchFindsKey` drives the full kernel against a known key.
- `TestGPUSearchCoversEveryKey` is the **anti-gap gate**, and matters specifically
  because of the group design: a kernel that skipped part of a block, mismapped an
  offset sign, or left a hole at a round boundary would still be fast and still pass
  the other two tests. It asserts every offset in a round is really found, and that
  consecutive rounds abut exactly. Re-run it after any change to `GRP_HALF`, the
  jump point, or the offset bookkeeping.

`search_test.go`'s `TestGeneratorHash160` anchors the CPU reference itself.

### Other design notes

- `walletNum` input is 1-indexed; it's converted to 0-indexed before array access.
- Field elements are four little-endian 64-bit limbs kept fully reduced in `[0, p)`;
  reduction uses `2^256 ≡ 2^32 + 977 (mod p)`. `__int128` intermediates are available in
  device code on this toolkit (CUDA 11.5, `sm_86`).

---
> Source: [lmajowka/cacagpu](https://github.com/lmajowka/cacagpu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
