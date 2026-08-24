## aec

> Pure-Go decoder for CCSDS 121.0-B-3 (Adaptive Entropy Coding). Decode-only, no cgo. Used by GRIB2 template 5.42, HDF5 szip, satellite telemetry. Module `github.com/shyrmapp/aec`, Go 1.26+. Single package, three files: `aec.go` (decoder, Params, flags), `bitreader.go`, `aec_test.go`.

# CLAUDE.md

Pure-Go decoder for CCSDS 121.0-B-3 (Adaptive Entropy Coding). Decode-only, no cgo. Used by GRIB2 template 5.42, HDF5 szip, satellite telemetry. Module `github.com/shyrmapp/aec`, Go 1.26+. Single package, three files: `aec.go` (decoder, Params, flags), `bitreader.go`, `aec_test.go`.

## Commands

```bash
go test ./...
go test -run TestCCSDS_AllOptions/p256n08 ./...        # one vector subtest
go test -bench=. -benchmem ./...
go test -run xxx -fuzz FuzzDecode -fuzztime 30s ./...  # also a CI smoke step
AEC_GRAEC=~/git/libaec/build/src/graec go test -run TestDifferentialGraec ./...  # skips without binary
go vet ./... && go build ./...
```

`TestDifferentialGraec` cross-checks 72 GRIB2-shaped param combos against the real libaec encoder. Trap: stock libaec's encoder never pads RSI (`ENABLE_RSI_PADDING` undefined) — `-p` sets the flag but emits unpadded streams; padded oracles need a build with `-DCMAKE_C_FLAGS=-DENABLE_RSI_PADDING`.

Test vectors in `testdata/` (CCSDS 121B2 official set, BSD-2): AllOptions n01–n32, LowEntropyOptions, ExtendedParameters. Tests `t.Skipf` on missing testdata — never fail hard. libaec default packing: 17–32-bit samples occupy 4 bytes each unless `-3` at encode time.

`testdata/ecmwf_aifs_t.aec` is a real ECMWF AIFS GRIB2 Section-7 payload; `TestRealWorldECMWFAIFS` compares output SHA256 against libaec 1.1.6. Ground truth for production traffic — if that hash changes, the decoder is wrong on real data regardless of synthetic vectors.

## Architecture

**API**: `Decode(data, Params)`, `DecodeInto(buf, data, Params)` (zero-alloc reuse), `NewDecoder(r, Params).DecodeAll()` (streaming). All funnel into `Decoder.DecodeAll`.

**Params** maps 1:1 to GRIB2 section 5 / szip metadata. `Flags` mirrors libaec `AEC_DATA_*`: supported `FlagMSB`, `FlagPreprocess`, `FlagPadRSI`; `FlagSigned`/`FlagRestricted` error by design; `Flag3Byte` accepted, no effect (output always `[]uint32`).

**Validation** rejects `NumValues<0`, `BitsPerSample` outside 0..32, `RSI<=0`, `BlockSize<=0` or odd. `RSI=0` previously infinite-looped — keep these checks.

**Decode loop**: `idLen` 3/4/5 bits from `BitsPerSample` (matches libaec). Outer loop RSIs (`FlagPadRSI`: byte-align before each RSI after the first). Inner loop blocks, per `id`:

- `id == 0` → low-entropy; sub-bit read **before** ref sample (libaec: m_id → m_low_entropy → m_low_entropy_ref). Sub-bit 0 = zero-block, 1 = second extension (`seTable`, size 91, CCSDS SE_TABLE).
- `id == (1<<idLen)-1` → no-compression, raw samples.
- else → split-sample, k = id-1; **all FS values first, then all k-bit remainders** (not interleaved).

First block of each RSI carries a reference sample **only when `FlagPreprocess`** (libaec gates `state->ref` on `state->pp`). After each RSI: `inversePreprocess` if preprocessing.

**bitReader**: MSB-first 64-bit accumulator; in-memory (`data`) or 64 KiB buffered stream (`r`). `fillUpTo` uses a single unaligned 64-bit load except near the tail. `readFS` uses `bits.LeadingZeros64` and **latches underflow when exhausted mid-FS** — without it a truncated stream of zeros fabricates a full "successful" decode. Streaming caps consecutive `(0, nil)` reads at 100. Underflow latches; `err()` returns `io.ErrUnexpectedEOF`; reads after underflow return zero — callers must check `err()`.

## Invariants when modifying

- Match libaec bit-stream order exactly (low-entropy sub-bit before ref; split-sample all-FS-then-remainders). Diverging breaks every vector.
- Reference samples exist **only with preprocessing** (`refPending := preprocess`); SE start index and zero-block `refSamples` gated the same. Treating every RSI as ref-bearing corrupts non-preprocessed streams (szip entropy-only) — `TestNonPreprocessed`, `TestDifferentialGraec`.
- Zero-block ROS=5 escape (`fs+1 == 5`): fill-to-boundary clamped by **both** `64 - blocksDecoded%64` and `RSI - blocksDecoded` (`TestZeroBlockROSBoundary`). `fs+1 > 5` decrements by one. Explicit zero run overrunning the RSI is a data error (`TestZeroBlockRSIOverrun`).
- SE branch: `i = 1` when `preprocess && blocksDecoded == 0` (ref slot already filled). Two samples per gamma; `d0` only on even `i`; `d1` dropped at `NumValues` boundary.
- Split-sample stores at most `NumValues - len(out)` samples but still consumes the full block's FS bits (encoder pads the last block) — keeps `DecodeInto` zero-alloc for non-aligned `NumValues` (`TestSplitOvershoot`).
- `inversePreprocess` is per-RSI, never across RSIs.
- All buffers `uint32`; signed support deliberately absent.
- Short reads latch underflow; `DecodeAll` returns `br.err()` **after** the loop — don't bail early, finishing keeps state consistent.

## Reference

Spec: https://public.ccsds.org/Pubs/121x0b3.pdf
libaec: https://github.com/MathisRosenhauer/libaec — when in doubt, cross-check `decode.c` (`m_id`, `m_split`, `m_low_entropy`, `m_se`, `m_zero_block`, `m_uncomp`).

The spec is the ultimate authority, but libaec can have bugs — and since nearly all real streams are libaec-encoded, a libaec quirk is de-facto standard. On spec/libaec conflict: follow libaec, document the divergence here; prefer the spec only when libaec's own decoder also rejects its encoder's output.

---
> Source: [shyrmapp/aec](https://github.com/shyrmapp/aec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
