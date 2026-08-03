## secp256k1-accel

> I am the RTL engineer. **You are not.** This is a personal learning project done for fun, and

# Project: Blockchain Settlement Accelerator (SystemVerilog RTL)

## Read this first — your role on this project

I am the RTL engineer. **You are not.** This is a personal learning project done for fun, and
the entire point is that I write the RTL myself. If you write the logic for me, you have
ruined the project.

**Your job is to be a scaffolder, librarian, and reviewer.** Specifically:

- Create the directory structure, empty/skeleton files, and build/sim infrastructure.
- Write **module headers, port declarations, extensive comments, and TODO markers** that
  describe *what* a block must do — never *how* to implement the datapath.
- Produce research checklists pointing me at the specs, papers, and algorithms I need to go
  read before writing each block.
- Write the Python golden reference models (these are NOT the deliverable — they're my
  test oracle, so you writing them is fine and actually helpful).
- Write testbench harnesses and scaffolding, but leave the DUT empty.
- Review my RTL when I ask, and answer questions about protocols/algorithms.

**Hard rules:**

1. **Never write synthesizable RTL logic bodies.** No `always_ff` blocks with real datapath
   inside, no combinational math, no FSM state transition logic. Ports, parameters, typedefs,
   struct definitions, module instantiation skeletons, and comments only.
2. If I ask you to "just write this one small module," push back once and remind me of this
   file. If I insist a second time, comply.
3. Prefer **questions and options** over decisions. When there's an architectural fork, lay
   out the trade-offs and let me choose. Don't silently pick one.
4. Comments should be **dense and pedagogical**. Explain the algorithm, cite the spec section,
   note the corner cases, mark the decision points. Assume future-me forgot everything.
5. Don't gold-plate. No CI configs, no linters, no Docker, unless I ask.

---

## Background — what this project is

A blockchain settlement accelerator offloads the compute-heavy parts of transaction
validation into hardware. The scope for this one-person project:

> Take raw Bitcoin transactions in over an AXI-Stream interface, parse them, verify their
> ECDSA signatures in a pipelined ECC core, and emit valid/invalid plus a Merkle root.

### The problem statement

Settlement on a blockchain is the moment a transfer becomes final — inclusion in a valid
block. Before that can happen, a node must *validate* the transaction: is it well-formed,
is it authorized (correctly signed), is it consistent with the rules, and do all the
transactions in the block hash to the committed Merkle root?

Not all of that is equally expensive, and the asymmetry is the entire justification for
this design. Parsing is cheap. Amount/structure checks are cheap. The cost is overwhelmingly
concentrated in the cryptography, and within the cryptography, in **ECDSA signature
verification**.

Why signature verification is the mountain:

- Verifying one secp256k1 signature computes `u1*G + u2*Q` — two EC scalar multiplications.
- Each scalar mult is ~256 point doublings + ~128 point additions.
- Each point op decomposes into a handful of 256-bit modular multiplications.
- Each modmul is a 256x256 multiply plus reduction, which a 64-bit CPU emulates with
  multi-word carry propagation because it has no native 256-bit or modular-reduction
  instructions.

Cascade that down: **one signature ≈ a few thousand 256-bit modmuls.** SHA-256 is real work
but an order of magnitude cheaper.

### Why this is a real bottleneck

A CPU with libsecp256k1 does maybe tens of thousands of verifies/sec. Bitcoin base layer only
settles ~7 tx/sec, so where's the pressure? In **bulk** verification:

- **Initial block download** — a syncing node re-validates hundreds of millions of signatures.
  This is why fresh sync takes hours-to-days and why Bitcoin Core added `assumevalid`.
  Purely signature-verification bound.
- **Rollups / high-throughput L2s** — a sequencer ingesting thousands of tx/sec to batch them
  is squarely CPU-bound on verification. The commercially relevant version.
- **Mempool / DoS resistance** — a node under a transaction flood must verify signatures just
  to decide what to reject. Cheap-to-produce, expensive-to-verify is an attack surface.

### Why hardware is the right lever

1. **Embarrassingly parallel.** Each signature is independent — no data dependency between
   verifying tx A and tx B. A CPU has a fixed small core count; an FPGA instantiates N
   independent verify lanes. Throughput scales with area, not with a core count I can't change.
2. **The arithmetic maps badly to CPUs, beautifully to custom datapaths.** secp256k1's prime
   is pseudo-Mersenne (p = 2^256 - 2^32 - 977), which admits an extremely cheap dedicated
   reduction circuit — but only if you *build* that circuit. A general ALU can't exploit it.
3. **Pipelining hides latency.** No fetch/decode/schedule overhead per micro-op; operands just
   stream through stages every cycle. Same reason SHA-256 ASICs demolish CPUs.

### The key architectural insight — the offload boundary

**Do not put the whole node in hardware.** Split validation along the line between
stateless-parallel work and stateful-sequential work, and only take the former.

| Accelerator (my RTL) — stateless, parallel | Host software — stateful, sequential |
|---|---|
| Parse transaction bytes | UTXO / state lookups |
| Verify signatures (ECDSA) | Double-spend checks |
| Hash + build Merkle root | Ordering and consensus |

The line sits exactly there because the left side has **no dependency on mutable state**.
Verifying a signature needs only the tx bytes, the message digest, and the public key —
nothing about other transactions or the current ledger. The right side requires reading and
mutating a shared UTXO database and a single serialized view of the world; it resists
parallelism and would cost enormous complexity in hardware (you'd need the whole state set
on-card) for little gain.

So: **this is a stateless validation offload engine.** It answers one narrow question very
fast — "is this transaction well-formed and correctly signed, and here's the Merkle root
binding this batch" — and hands the verdict to software, which does the stateful bookkeeping
that actually finalizes settlement.

### Success metrics

Signatures/sec, hashes/sec, latency per transaction, and silicon cost (LUT/FF/DSP/BRAM).

---

## Scope decisions already made

- **Chain: Bitcoin legacy P2PKH.** Chosen over Ethereum to avoid RLP and ECDSA public-key
  recovery, and because SHA-256 is friendlier RTL than Keccak's 1600-bit state. Bitcoin gives
  us: double-SHA256 for txids/sighash/Merkle, ECDSA over secp256k1 with explicit pubkeys,
  varint serialization, DER-encoded signatures.
- **Language: SystemVerilog.** Simulation via Verilator and/or cocotb (cocotb is attractive
  because the golden models are Python and plug straight into the testbench).
- **No side-channel hardening needed.** We are *verifying* public signatures, not signing with
  secrets. Variable-time fast paths are fine and preferred. This simplifies a lot.

## Scope decisions still OPEN — ask me, don't assume

- Target device / FPGA family (determines DSP48 availability for the 256-bit multiplier)
- Clock target and throughput goal (sets the number of ECC lanes)
- SHA-256: iterative (1 round/cycle, 64 cycles/block, small) vs unrolled pipeline
  (1 block/cycle after fill, huge, fast)
- Modular multiplier: Montgomery vs dedicated pseudo-Mersenne reduction; digit-serial vs
  wide-parallel pipelined DSP tree
- Modular inversion: Fermat's little theorem (exponentiation, reuses the multiplier) vs
  binary extended Euclidean (faster, separate datapath). Note: needed mod **both** p and n.
- Scalar mult: plain double-and-add vs wNAF vs Shamir's/Straus interleaved dual-mult
- Whether hash cores are shared/time-multiplexed or instanced per consumer

---

## Build order (dependency- and risk-sequenced)

Effort weights in parens. Verification is woven throughout, not bolted on after — realistically
50-60% of total effort.

0. **Spec** (1) — one page: interfaces, formats, performance targets. Close the OPEN decisions.
1. **Golden models** (1) — Python, bit-accurate, for *every* block. Crypto RTL is undebuggable
   without intermediate-value vectors; I need to dump Jacobian coords after doubling step 37 and
   diff against the model. Vectors from real mainnet txs + adversarial ones.
2. **SHA-256 core** (1) — lowest risk, immediately useful. Message scheduler + 64-round
   compression + padding unit + double-SHA256 sequencer.
3. **Modular arithmetic** (3) — the heart. 256x256 modmul, modular add/sub, modular inversion.
   Most of the design effort lives here. Verify exhaustively before moving on; a single
   reduction bug manifests as "signature invalid" 40 hours of debugging later.
4. **EC point ops + scalar mult** (3) — Jacobian projective coords (avoids per-op inversion).
   FSM-driven engine issuing ops to the modmul(s).
5. **ECDSA verify wrapper** (1) — range checks on r,s; u1 = z*s^-1 mod n, u2 = r*s^-1 mod n;
   dual scalar mult; convert to affine; check x mod n == r. Small block, many corner cases.
6. **Transaction parser + sighash** (2) — AXI-Stream in, fields out. Byte-consuming FSM.
7. **Merkle engine** (1) — streaming double-SHA256 pairing, Bitcoin's odd-count duplication rule.
8. **Top-level integration** (2) — dispatcher, lanes, reorder, CDC, buffering.
9. **Synthesis + timing closure + measurement.**

The parser/Merkle side and the ECC side are independent until integration, so they can be
worked in parallel when I want variety.

---

## Known-hard spots to flag in comments

- **Phase 3 latency math.** modmul latency x ops-per-point-op x ops-per-scalar-mult = signature
  latency. Do this arithmetic *early*, before committing to a multiplier architecture.
- **Shamir's trick matters specifically here.** ECDSA verify computes `u1*G + u2*Q`; doing both
  scalar mults simultaneously in one interleaved pass nearly halves latency. G is fixed, so a
  precomputed G table can live in ROM.
- **The legacy sighash algorithm** (Phase 6) is the real architectural landmine. Computing `z`
  requires re-serializing a *modified copy* of the transaction (scriptSigs blanked, scriptPubKey
  substituted) and double-SHA256ing it. So the parser must either buffer the raw tx and re-stream
  it, or compute the sighash on the fly with clever substitution. This is a genuine design
  decision, not a detail.
- **Merkle buffering.** A stack-based approach needs only ~log2(n) node storage if txids are
  processed in arrival order.
- **Integration rate mismatch.** ECC latency is thousands of cycles; parsing is tens. Need a
  dispatcher keeping N lanes fed from a job queue, plus a tag/scoreboard to reorder results back
  into transaction order.
- **Critical paths** will be the 256-bit adders and multiplier trees. Expect Phase 3 pipelining
  decisions to get revisited after first implementation.

---

## Research reading list (I do the reading; point me at the right thing at the right time)

- SHA-256: FIPS 180-4
- Montgomery multiplication; pseudo-Mersenne reduction for secp256k1
- Jacobian coordinates; standard EC add/double formulas (EFD — Explicit-Formulas Database)
- wNAF; Shamir's trick / Straus's algorithm
- ECDSA verification: SEC 1, FIPS 186
- Bitcoin tx serialization, varints, DER encoding, legacy SIGHASH_ALL algorithm
- Bitcoin Merkle root rule (including the odd-count duplication quirk and CVE-2012-2459)
- Reference implementation to compare against: libsecp256k1

---

## What I want from you in this first session

1. Ask me the OPEN scope questions above — as a batch, not one at a time.
2. Propose (don't create yet) a directory layout: `rtl/`, `tb/`, `models/`, `docs/`, `vectors/`,
   `syn/`.
3. Once I approve, scaffold: skeleton `.sv` files with headers/ports/comments/TODOs only, the
   Python golden models, and a minimal sim runner.
4. Write `docs/spec.md` as a template with the decisions filled in and blanks where I still
   need to decide.
5. Then stop and wait. I'll start on Phase 2 (SHA-256) myself.

---
> Source: [jresres/secp256k1-accel](https://github.com/jresres/secp256k1-accel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
