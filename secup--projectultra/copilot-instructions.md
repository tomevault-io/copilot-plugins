## projectultra

> **If this is a new/fresh session, do this FIRST before any work:**

# ProjectUltra - HF Sound Modem

## FRESH SESSION? START HERE

**If this is a new/fresh session, do this FIRST before any work:**

1. **Read AI collaboration playbook:** `cat docs/AI_COLLABORATION.md` - **MANDATORY** - How to work with Codex (the other AI on this project), when to involve it, brief format, verification gates, autonomous-mode rules
2. **Read project goals:** `cat docs/PROJECT_GOALS.md` - Mission, priorities, and task filter
3. **Read current agent/project state:** `cat docs/AGENT_CURRENT_STATE.md` - Current automation and handoff context
4. **Check known bugs:** `cat docs/KNOWN_BUGS.md` - Active bugs you must not re-discover
5. **Check recent changes:** `git log --oneline -10` - See recent commits

**Before modifying ANY code, read:**
- `docs/AI_COLLABORATION.md` - Required workflow with Codex for non-trivial changes
- `docs/PROJECT_GOALS.md` - Mission, priorities, throughput/reliability targets, and agent task rule
- `docs/INVARIANTS.md` - Critical rules that MUST NOT be violated (causes subtle bugs if ignored)

**Autonomous agent work:**
- Use `docs/AGENTIC_DEVELOPMENT.md` and one task file in `agents/queue/`; do not work from an open-ended prompt.
- Use `docs/PROJECT_GOALS.md` to keep work aligned with the modem mission and current priorities.
- Use `docs/AGENT_TASK_BACKLOG.md` for approved task candidates and acceptance criteria.
- Use `docs/AGENT_DEDICATED_ENV_MACOS.md` for MacBook dedicated-agent setup.
- Use `docs/AGENT_CURRENT_STATE.md` to recover compacted/lost agent-system context.
- Run `./agents/run_local_gate.sh` before claiming a task is done.
- Use `./agents/run_hardware_smoke.sh` for PHY/ARQ/audio-path changes and respect the hardware lock.
- Do not grant agents unrestricted shell access; use repo-scoped allowlists from `agents/permissions/`.

**This project has durable documentation files.** They exist because context was lost repeatedly, causing rework. USE THEM.

---

## Hardware Audio Calibration - Mac <-> Pi 5 Test Rig

**Current known-good calibration (2026-04-29):**
- Mac USB soundcard: `Sound Blaster Play! 3`
- Pi USB soundcard: ALSA card 0, `USB Audio Device`
- Mac volume: output `71`, input `60`
- Pi mixer: `Speaker` 65% (`-13.00 dB`), `Mic Capture` 57% (`+8.00 dB`), `Auto Gain Control` off
- Synthetic channel hardware tests: use `--inject --inject-gain 0.70`

**Apply calibration before hardware tests:**
```bash
osascript -e 'set volume input volume 60' -e 'set volume output volume 70'
ssh -i "$HOME/.ssh/id_pi5" math@pi5tester \
  "amixer -D default sset 'Auto Gain Control' off && \
   amixer -D default sset 'Speaker' 65% && \
   amixer -D default sset 'Mic' capture 57%"
```

**Verify raw audio before modem tests:**
```bash
SSH_KEY="$HOME/.ssh/id_pi5" ./tools/check_hw_audio_path.sh
```

Expected calibrated raw levels:
- Pi -> Mac: RMS about `0.124`, peak about `0.303`, per-channel rough frequency near `1 kHz`
- Mac -> Pi: RMS about `0.249`, peak about `0.408`, per-channel rough frequency near `1 kHz`
- Acceptable target: RMS `0.05-0.25`, peak `0.15-0.80`
- Too hot: peak above `0.90`; this risks ADC/DAC clipping and invalid fading-test results
- Too low/silent: RMS near `0.0003` or below; check cable/device selection

Last verified captures:
- `/tmp/ultra_audio_path_20260429_220218/pi_to_mac_capture.wav`
- `/tmp/ultra_audio_path_20260429_220218/mac_to_pi_capture.wav`

Last post-calibration modem sweep:
- Good injected, 1 KB, R1/2, SNR 20/15/12: pass, `0` retx, `/tmp/ultra_hw_20260429_220250`, `/tmp/ultra_hw_20260429_220341`, `/tmp/ultra_hw_20260429_220445`
- Moderate injected, 1 KB, R1/2, SNR 20/15/12: pass with `4/7/8` retx, `/tmp/ultra_hw_20260429_220538`, `/tmp/ultra_hw_20260429_220717`, `/tmp/ultra_hw_20260429_220828`
- Moderate injected, 1 KB, R1/4, SNR 15/12: pass with `8/8` retx, `/tmp/ultra_hw_20260429_221117`, `/tmp/ultra_hw_20260429_221240`
- Good injected, 5 KB, R1/2, SNR 15: pass with `13` timeout retx, `/tmp/ultra_hw_20260429_221423`

Post ACK/control robustness patch checks:
- AWGN injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_222350`
- Good injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_222920`
- Moderate injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_223017`
- Moderate injected, 1 KB, R1/4, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_223253`
- Moderate injected, 1 KB, R1/2, SNR 12: pass with `7` retx, `/tmp/ultra_hw_20260429_223113`
- Moderate injected, 1 KB, R1/4, SNR 12: pass with `5` retx, `/tmp/ultra_hw_20260429_223754`
- Good injected, 5 KB, R1/2, SNR 15: pass with `4` retx, `/tmp/ultra_hw_20260429_223926`

Corrected two-sided Pi/Mac rebuild checks:
- Good injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_224520`
- Moderate injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_224612`
- Good injected, 5 KB, R1/2, SNR 15: pass with `4` timeout retx, `/tmp/ultra_hw_20260429_224658`; BRAVO failed the original seq32-35 data burst (`CW[0..3]: FAIL`) and decoded the retransmissions, so this is data-side loss, not ACK/control loss.

Final ACK/control + burst/data-acquisition checks:
- AWGN injected, 1 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_230135`
- Good injected, 5 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_225150`
- Moderate injected, 5 KB, R1/2, SNR 15: pass, `0` retx, `/tmp/ultra_hw_20260429_225916`
- Moderate injected, 1 KB, R1/2, SNR 12: pass, `0` retx, `/tmp/ultra_hw_20260429_225822`

Interpretation of the 2026-04-29 robustness work:
- ACK/control decode is healthy in AWGN, Good SNR15, Moderate SNR15, and the SNR12 Moderate canary: cumulative ACKs repeat when profile ACK diversity is enabled, and the 1-CW control LLR gate admits real fading ACKs down to `|LLR|_avg ~= 1.5`.
- The 5 KB Good residual was a burst-interleaver receiver bug, not LDPC weakness: a physical burst block at `RMS=0.0390` was below the old hard `0.0400` gate, aborting the whole 4-frame group. The decoder now demodulates weak blocks down to `0.015` and only inserts zero-LLR erasures below that, so one weak physical block does not become four ARQ retransmissions.
- The SNR12 Moderate tail retry was a data acquisition gate issue: real tail DATA can arrive around `corr ~= 0.52-0.56` and `|LLR|_avg ~= 1.7`. Connected DQPSK data sync and 4-CW escalation now admit those candidates while the false-lock LLR/near-zero gates still reject obvious noise.

**Important distinction:** hardware gain staging does not replace injected-channel headroom. The Watterson injector can generate samples above full scale before the soundcard. Keep `--inject-gain 0.70` unless a new calibration sweep proves a different value.

---

## CRITICAL RULES (Never Violate)

**Test binaries:**
- ALWAYS run from `build/` directory: `./build/cli_simulator`, NOT `./cli_simulator`
- `cli_simulator` - **PRIMARY** test tool for full protocol with light preamble in connected mode
  - Tests: PING/PONG → CONNECT → MODE_CHANGE → DATA (4CW frame interleaved) → DISCONNECT
  - Command: `./build/cli_simulator --snr 15 --fading good --rate r1_4 --test 2>&1 | tee /tmp/test_output.log`
- `test_waveform_simple` - Quick single-frame sanity checks (NOT for connected mode testing)
- `ctest --test-dir build` - default maintained unit/regression gate
- `tests/regression_matrix.sh` - wrapper for default CTest; `--full` also runs maintained light-sync sweep when `cli_simulator` exists
- Obsolete direct-modem/QPSK harnesses were removed from `tests/`; use Git history for archaeology

**MC-DPSK invariants:**
- ALWAYS call `mc_dpsk_demodulator_->reset()` at start of `rxDecodeDPSK()`
- ALWAYS call `setCFO(frame.cfo_hz)` to reset CFO accumulation between frames
- CFO from chirp detection is TRUSTED over training-based CFO estimation

**OFDM CFO invariants (see `docs/CFO_CORRECTION_FLOW.md` for full details):**
- Chirp CFO is coarse — on fading channels it can be wrong by ±2 Hz
- LTS residual CFO estimation REFINES chirp CFO per frame (threshold 0.3 Hz)
- CFO feedback loop: after demodulation, corrected CFO propagates back to cached `last_cfo_`
- NEVER remove the feedback in `ofdm_chirp_waveform.cpp:process()` or `streaming_decoder.cpp`
- Without feedback, wrong chirp CFO re-injects on every frame → progressive phase drift → CW failures

**Testing invariants:**
- Use SINGLE ModemEngine instance for entire audio stream (continuous RX)
- Buffer limit: MAX_PENDING_SAMPLES = 960000 (20 seconds at 48kHz)
- **DEFAULT unit/regression gate:** `ctest --test-dir build --output-on-failure -j4`
- **PRIMARY regression test:** `./build/cli_simulator --snr 15 --fading good --rate r1_4 --test 2>&1 | tee /tmp/test_output.log`
- `cli_simulator` is the ONLY tool that tests the full protocol with light preamble, two-station interaction, and proper connected-mode configuration. `test_waveform_simple` is for quick single-frame sanity checks only.
- **ALWAYS use `| tee /tmp/test_output.log`** when running tests - tests take minutes and we need full output for debugging

---

## TWO OPERATING MODES - CRITICAL ARCHITECTURE

**The modem has TWO completely different operating modes based on SNR:**

### Mode 1: MC-DPSK (SNR ≤ 10 dB)
- **When:** SNR below 10, or heavy fading conditions
- **Waveform:** Multi-Carrier DPSK with chirp sync
- **ARQ:** Stop-and-wait (window=1) - send ONE frame, wait for ACK
- **Frame format:** Variable codewords, simple sequential encoding
- **Control frames:** 20 bytes (ACK, NACK, etc.) - NO patching, encode as-is
- **Data frames:** Variable CWs based on payload size
- **Interleaving:** NONE (no frame interleaving, no channel interleaving)
- **Preamble:** ALWAYS full chirp preamble (no light sync)
- **Key files:** `decodeMCDPSKFrame()` in streaming_decoder.cpp

### Mode 2: OFDM (SNR ≥ 10 dB)
- **When:** SNR 10 and above with acceptable fading
- **Waveform:** OFDM with chirp or Schmidl-Cox sync
- **ARQ:** Selective Repeat (window=8) - send up to 8 frames before waiting
- **Frame format:** Fixed 4-codeword frames for data, 1-codeword for control
- **Control frames:** 20 bytes, 1 CW, no frame interleaving (fast ACK)
- **Data frames:** 4 CWs with frame interleaving
- **Interleaving:** Frame-level interleaving (spreads CWs), optional channel interleaving
- **Preamble:** Light preamble (LTS only) for data after handshake
- **Key files:** `decodeFixedFrame()` in frame_v2.cpp

### Mode 3: OFDM_NARROW (SNR 5-10 dB, 500 Hz bandwidth)
- **When:** Narrowband chirp detected (1250-1750 Hz), low SNR where wideband fails
- **Waveform:** OFDM with narrowband chirp sync, FFT=2048, 21 carriers, 492 Hz BW
- **ARQ:** Stop-and-wait (window=1) - same as MC-DPSK due to long frame times (~3s)
- **Frame format:** Same as wideband OFDM (4 CW data, 1 CW control)
- **Interleaving:** Frame-level + optional channel interleaving (same as OFDM)
- **Preamble:** Light preamble (LTS only) for data after handshake
- **Dual-listen:** RX listens for both wideband AND narrowband chirps when idle
- **Throughput:** ~103 bps (R1/4) to ~230 bps (R1/2) — 10× slower than wideband but works at 7.5 dB lower SNR

### Mode Selection Flow
1. **Connection starts:** MC-DPSK for PING/PONG/CONNECT (wideband or narrowband chirp)
2. **Dual-listen:** RX detects chirp type → sets BandwidthMode (WIDE or NARROW)
3. **After CONNECT_ACK:** SNR is measured, mode is negotiated
   - SNR < 10 + wideband: Stay in MC-DPSK
   - SNR ≥ 10 + wideband: Switch to OFDM_CHIRP
   - Narrowband detected: Switch to OFDM_NARROW
4. **enterConnected():** Sets ARQ window based on mode (1 for MC-DPSK/NARROW, 4 for wideband OFDM)
5. **StreamingEncoder/Decoder:** Check `mode_` to use correct path, `isOFDMMode()` for OFDM family

### NEVER MIX THESE:
- MC-DPSK frames through OFDM encoder (corrupts control frames)
- OFDM interleaving on MC-DPSK data
- Light preamble with MC-DPSK
- Window > 1 with MC-DPSK

---

**Performance Requirements (cli_simulator --test):**
| Mode | Channel | SNR | Required CW Success |
|------|---------|-----|---------------------|
| MC-DPSK | AWGN | 5+ | 100% |
| MC-DPSK | Moderate fading | 10 | 100% |
| OFDM_CHIRP | AWGN | 15+ | 100% |
| OFDM_CHIRP | Good fading | 15 | **100%** |
| OFDM_CHIRP | Moderate fading | 15 | ~90% |
| OFDM_NARROW | AWGN | 8+ | 100% |
| OFDM_NARROW | Good fading | 8 | 100% data, 90%+ ACK |

**Current state (2026-03-15):**
- MC-DPSK: WORKING - 100% at SNR=10 with moderate fading
- OFDM_CHIRP DQPSK R1/4 AWGN: WORKING - 100% at SNR=15 and SNR=20 (0 retries)
- OFDM_CHIRP DQPSK R1/4 Good fading SNR=15: WORKING - 100% (0 retries, 0 failures)
- OFDM_CHIRP DQPSK R1/4 Good fading SNR=10: WORKING - 30/30 seeds PASS (avg 1.5 retx, 100% delivery)
- OFDM_CHIRP DQPSK R1/4 Moderate fading SNR=15: WORKING - 5/5 seeds PASS (avg 1.4 retx, 100% delivery)
- OFDM_CHIRP DQPSK R1/2 AWGN: WORKING - 100% at SNR=15 and SNR=20 (0 retries)
- OFDM_CHIRP DQPSK R1/2 Good fading: WORKING - 100% at SNR=15 (5/5 seeds, 0 retries)
- OFDM_CHIRP DQPSK R1/2 Moderate fading SNR=15: WORKING - 5/5 seeds PASS (avg 2.4 retx, 100% delivery)
- OFDM_CHIRP DQPSK R2/3 AWGN: WORKING - 100% at SNR=20 (0 retries)
- OFDM_CHIRP DQPSK R2/3 Good fading SNR=20: WORKING - 30/30 seeds PASS, 0 retransmissions
- OFDM_CHIRP DQPSK R2/3 Good fading SNR=15: WORKING - 10/10 seeds PASS (avg 1.5 retx, 100% delivery)
- OFDM_CHIRP QPSK R1/2 AWGN: WORKING - 100% at SNR=20 (0 retries)
- OFDM_CHIRP QPSK R1/2 Good fading: WORKING - avg 95% frame success at SNR=20 (30-seed survey, all messages delivered via ARQ)
- OFDM_CHIRP QPSK R2/3 AWGN: WORKING - 100% at SNR=20 (0 retries)
- OFDM_CHIRP QPSK R2/3 Good fading: WORKING - 5/5 seeds PASS (2 seeds had retx, 3 clean)
- OFDM_CHIRP DQPSK R3/4 AWGN: WORKING - 100% at SNR=20 (10/10 seeds, 0 retries)
- OFDM_CHIRP DQPSK R3/4 Good fading: NOT RECOMMENDED (23 retx / 5 seeds — AWGN only)
- 1-CW ACK frames: WORKING - control frames use 1 CW (3x faster ACK)
- Variable-CW frames: WORKING - CONNECT/DISCONNECT use exact CW count (2 at R1/2, 3 at R1/4)
- OFDM_NARROW DQPSK R1/4 AWGN: WORKING - 100% at SNR=8 (0 retransmissions)
- OFDM_NARROW DQPSK R1/4 Good fading SNR=8: WORKING - 100% data decode, 93% ACK, all messages delivered via ARQ
- OFDM_COX: WORKING - DATA phase passes at SNR=20 dB
- OTFS/MFSK: RESERVED ONLY - not in the production build or default capabilities
- cli_simulator: FULLY WORKING - all phases pass on AWGN and fading

**Auto rate selection ladder (2026-03-15, updated with 802.11n LDPC):**
| Condition | Auto rate | Payload/frame | Throughput |
|-----------|-----------|---------------|------------|
| SNR >= 20, AWGN (fading < 0.15) | **R3/4** | 243 bytes | ~3900 bps |
| SNR >= 15, near-AWGN (fading < 0.15) | **R2/3** | 197 bytes | ~3200 bps |
| SNR >= 15, good/moderate fading (< 1.10) | **R1/2** | 141 bytes | ~2300 bps |
| Everything else | **R1/4** | 62 bytes | ~1150 bps |

**Temporal fading measurement (2026-02-03):**
- `getFadingIndex()` now combines freq_cv (multipath) + temporal_cv (Doppler spread)
- temporal_cv measures per-carrier magnitude variance over ~40+ symbols (~0.4s)
- Good channels (0.1Hz Doppler) show low temporal_cv (~0.03-0.30)
- Moderate channels (0.5Hz Doppler) show high temporal_cv (~0.40-0.55)
- **Trailing silence bug found and fixed**: `demodulateSoft()` was demodulating ~9 silence symbols
  at end of long frames (131 symbols, only 122 valid). This caused temporal_cv=0.27 on AWGN.
  Now detects and excludes trailing silence using energy-based threshold (20% of reference).
- Calibrated combined fading index values:
  - AWGN: ~0.04 (freq_cv ~0.003, temporal_cv ~0.032)
  - Good fading: ~0.62 (freq_cv ~0.32, temporal_cv ~0.30)
  - Moderate fading: ~0.90 (freq_cv ~0.42, temporal_cv ~0.49)
  - Poor fading: ~0.82 (freq_cv ~0.33, temporal_cv ~0.49)
- All `isFading()` thresholds updated from 0.4 to 0.75 across codebase
- Waveform selection thresholds recalibrated: AWGN < 0.15, Good < 0.75, Moderate < 1.10
- OFDM internal fading thresholds (LLR scaling, two-pass) NOT changed — they use
  pilot-variance-based `last_fading_index` on a separate scale

**Fading channel notes (2026-03-15):**
- Fading index now combines freq_cv + temporal_cv for better Good vs Moderate separation
- OFDM internal uses separate `last_fading_index` from pilot variance (~0.15-0.50)
- LLR scaling (1 + 10×fading²) applied when OFDM fading_index > 0.15 to prevent overconfident wrong bits
- Two-pass DQPSK decoding enabled for fading channels (threshold=0.12)
- Two-pass uses `last_fading_index` from pilot variance (NOT computeFadingIndex() which returns 0 after sync)
- Light sync confidence threshold=0.8 (raised from 0.5) to reject marginal timing syncs on fading channels
- CFO drift limited to ±1 Hz per frame when connected (multipath can cause false CFO readings)
- **CFO feedback loop** (2026-02-03): Pilot-corrected CFO propagates back to StreamingDecoder's cached value
- **LTS residual CFO** (2026-02-03): Detects and corrects chirp CFO errors >0.3 Hz from training symbols
- **CPE correction for differential modes** (2026-03-15): Per-symbol common phase error tracking
  now enabled for DQPSK/D8PSK (was coherent-only). Estimates phase drift from pilots each symbol,
  clamps to ±15° for differential (prevents overcorrection from noisy fading pilots).
  This keeps channel_estimate phase tracking the actual channel, improving MMSE equalization.
  Safe for DQPSK: CPE cancels in diff = eq[n] × conj(eq[n-1]) since both use same corrected H.
- Good fading: **100% CW success** at R2/3 SNR=15 (10/10 seeds, enabled by CPE correction)
- Moderate fading: R1/4 avg 1.4 retx (5/5 seeds), R1/2 avg 2.4 retx (5/5 seeds) — up from ~89% CW

**OTFS/MFSK production status (2026-04-30):**
- OTFS prototype code was removed from the production build. The tested implementation was not competitive with OFDM_CHIRP on fading and required research-grade DD-domain equalization to justify keeping it.
- MFSK enum/capability values are reserved for wire compatibility, but there is no maintained production implementation. Do not advertise or negotiate MFSK.
- Default `ModeCapabilities::ALL` includes only production-supported modes: OFDM_COX, MC_DPSK, OFDM_CHIRP, and OFDM_NARROW.

**Recommendation:** Use OFDM_CHIRP with DQPSK. Rate selection is automatic via `selectOFDMCodeRate()`:
- R3/4 for AWGN only (SNR≥20, fading<0.15) — ~3.4× throughput vs R1/4
- R2/3 for good fading or better (SNR≥15) — ~2.8× throughput vs R1/4
- R1/2 for good/moderate fading (SNR≥15) — ~2× throughput vs R1/4
- R1/4 for heavy fading or lower SNR — robust but slower

**FFTW requirement:**
- FFTW3 is REQUIRED for fast chirp detection (apt install libfftw3-dev)
- Without FFTW: Cooley-Tukey fallback takes ~1 second per correlation (unusable)
- With FFTW: Detection takes ~0.5s after frame TX ends (correct)
- FFTW planner is NOT thread-safe - protected by global mutex in fft.cpp

**StreamingDecoder design:**
- Expects audio fed at real-time rate (or close to it)
- Has overflow protection if audio fed faster than processing
- Mode switches reset correlation_pos_ to start searching new data

**Pending Improvements (2026-02-03):**
- **CFO Pre-Correction for LTS Sync:** Light preamble LTS sync detection still happens BEFORE CFO
  correction in StreamingDecoder. The LTS residual fix (in demodulator) corrects CFO after sync,
  but pre-correcting audio samples in StreamingDecoder would improve LTS detection reliability
  on fading+CFO channels. Cost: O(N) complex multiply per sample - very cheap.
- **Per-Symbol Pilot Tracking:** IMPLEMENTED and active in `channel_equalizer.cpp:424-694`.
  Every data symbol updates H via LS pilot estimation, alpha-smoothed tracking (alpha=0.8
  for differential w/ pilots), residual CFO tracking, and timing recovery from pilot phase slope.
  Pilots spaced every 10 carriers (~6 pilots across 59 carriers). R1/2 still struggles on
  moderate fading due to insufficient LDPC redundancy, not missing pilot tracking.

---

## Important Rules

- **No guessing.** If you don't know an answer, say "I don't know" or "I need to verify X" — don't fabricate. Read the code, run the test, or ask the user. Be direct. Short answers > long ones with hedging. Length is not a proxy for quality.

- **Never mention specific competing products by name** (e.g., no "VARA", "ARDOP", "Winlink" etc.). Always refer to "industry leaders", "commercial HF modems", or "existing systems" instead.

- **MANDATORY: Document ALL fixes and changes** in `docs/CHANGELOG.md` with what was broken, what changed, how it's fixed, and test verification.

- **Track bugs in KNOWN_BUGS.md.** Add with unique ID (BUG-XXX). When fixed, move to "Fixed Bugs" section.

- **Read INVARIANTS.md before changing critical code.** Violating these causes subtle bugs.

- **Follow `docs/QUALITY_STRATEGY.md` for Tier 0/Tier 1 changes.** Do not chase fake coverage. Add meaningful tests for modem-critical behavior, remove stale code, and keep coverage gates reproducible.

- **Run the local quality gate before committing critical code:** `cmake --build build -j4 && ctest --test-dir build --output-on-failure -j4 && ./tests/regression_matrix.sh --quick && ./scripts/coverage_report.sh`

---

## Essential Documentation

### Priority 1: Always Read First
| Document | Purpose |
|----------|---------|
| `docs/PROJECT_GOALS.md` | Mission, priorities, throughput/reliability targets, and agent task filter |
| `docs/AGENT_CURRENT_STATE.md` | Current agent-system state and compact handoff |
| `docs/QUALITY_STRATEGY.md` | Critical-software testing, coverage, CI, and refactor policy |
| `docs/QUALITY_AUDIT.md` | Current quality baseline, coverage gaps, and hardening backlog |
| `docs/KNOWN_BUGS.md` | Active bugs - DON'T rediscover these |
| `docs/INVARIANTS.md` | 25 critical rules that MUST NOT be violated |
| `docs/ALPHA_RELEASE_GATE.md` | Release criteria and seeded gate commands |
| `docs/CHANGELOG.md` | History of all fixes - DON'T redo this work |

### Priority 2: Read When Working on Subsystem
| Document | Purpose |
|----------|---------|
| `docs/CFO_CORRECTION_FLOW.md` | **CRITICAL** - 4-stage CFO flow, fading fix, feedback loop |
| `docs/PROTOCOL_V2.md` | Frame formats, protocol flow |
| `docs/GUI_ARCHITECTURE.md` | ImGui widgets, threading, virtual station |
| `docs/AUDIO_SYSTEM.md` | SDL2 audio I/O, buffers, latency |
| `docs/CONFIGURATION_SYSTEM.md` | AppSettings, ModemConfig, presets |
| `docs/BUILD_SYSTEM.md` | CMake, dependencies, tests, coverage script |
| `docs/AGENTIC_DEVELOPMENT.md` | Bounded agent workflow, permissions, gates, and review process |

### Priority 3: Reference
| Document | Purpose |
|----------|---------|
| `docs/BUILD_SYSTEM.md` | CMake, dependencies, adding components |
| `docs/ADDING_NEW_WAVEFORM.md` | Step-by-step guide for adding future waveform implementations |
| `docs/GIT_WORKFLOW.md` | Commit strategy, branching, push policy |
| `docs/README.md` | Documentation map; archive docs are historical only |

---

## Quick Reference

### Build
```bash
mkdir build && cd build
cmake ..
make -j4
```

### GUI Application
```bash
./ultra_gui              # Normal mode
./ultra_gui -sim         # Developer mode with virtual station
./ultra_gui -sim -rec    # With audio recording
```

### Test Tools
| Tool | Purpose | Example |
|------|---------|---------|
| `cli_simulator` | **PRIMARY** - Full protocol with two-station interaction | `./build/cli_simulator --snr 15 --fading good --rate r1_4 --test` |
| `test_waveform_simple` | Quick single-frame sanity check | `./build/test_waveform_simple -w ofdm_chirp --snr 15` |
| `tests/regression_matrix.sh` | Maintained CTest wrapper | `./tests/regression_matrix.sh --quick` |

**Testing priority:** Use CTest for unit/regression gates and `cli_simulator` for full real testing (handshake, light preamble, data transfer, ARQ). Use `test_waveform_simple` only for quick single-frame sanity checks.

### CLI Commands
```bash
# Send PING probe
./ultra ptx ping -s MYCALL | aplay -f FLOAT_LE -r 48000

# Send message
./ultra ptx "Hello World" -s MYCALL -d THEIRCALL -o msg.f32

# Decode recording
./ultra prx recording.f32           # OFDM
./ultra prx -w dpsk recording.f32   # DPSK/PING
```

---

## Waveform Summary

| Mode | Sync Method | SNR Range | Max Throughput | CFO Tolerance | Fading |
|------|-------------|-----------|----------------|---------------|--------|
| **MC-DPSK** | Dual Chirp | -3 to 10 dB | 938 bps | ±50 Hz | Good |
| **OFDM_NARROW** | NB Chirp + LTS | 5-10 dB | ~230 bps | ±50 Hz | Good (R1/4) |
| **OFDM_CHIRP** | Dual Chirp + LTS | 10-17 dB | 3.4 kbps | ±50 Hz | Good (R1/4) |
| **OFDM_COX** | Schmidl-Cox | 17+ dB | 7.9 kbps | Needs testing | Poor |
| **SC-DPSK** | Barker-13 | -8 to -3 dB | 125 bps | N/A | Good |

**Waveform Selection:**
- Poor HF channels (2ms delay): Use MC-DPSK
- Low SNR (5-10 dB) with good/moderate fading: Use OFDM_NARROW (~103 bps R1/4, ~230 bps R1/2)
- Moderate/Good HF: Use OFDM_CHIRP (10-17 dB) or OFDM_COX (17+ dB)
- Very low SNR: Use SC-DPSK or MC-DPSK
- OTFS/MFSK values are reserved only and are not production-supported

---

## Architecture Overview

```
src/
├── gui/modem/          # ModemEngine - TX/RX audio processing
├── ofdm/               # OFDM modulator/demodulator
├── psk/                # Single/Multi-carrier DPSK
├── fec/                # LDPC encoder/decoder (648-bit codewords)
├── protocol/           # Protocol v2 (PING/CONNECT/DATA/DISCONNECT)
├── sync/               # ChirpSync, Schmidl-Cox sync
└── waveform/           # IWaveform interface and implementations

tools/
├── cli_simulator.cpp   # Full protocol test
└── test_waveform_simple.cpp # Quick waveform sanity checks
```

**Key Files:**
- `src/sync/chirp_sync.hpp` - Dual chirp detection + CFO estimation
- `src/gui/modem/modem_rx_decode.cpp` - RX decode logic
- `src/psk/multi_carrier_dpsk.hpp` - MC-DPSK modulator/demodulator

---

## Key Specifications

| Parameter | Standard Mode | NVIS Mode |
|-----------|---------------|-----------|
| Sample Rate | 48,000 Hz | 48,000 Hz |
| Center Frequency | 1,500 Hz | 1,500 Hz |
| Bandwidth | ~2.8 kHz | ~2.8 kHz |
| FFT Size | 512 | 1024 |
| Carriers | 30 | 59 |
| Max Throughput | 3.4 kbps | 7.2 kbps |

---

## Known Limitations

1. **OFDM_COX CFO:** Needs verification at 17+ dB with Schmidl-Cox
2. **Poor HF channels (2ms delay):** OFDM fails - use MC-DPSK instead
3. **MC-DPSK floor:** -5 dB is hard floor (20-40% success)
4. **File transfer:** DATA_START/DATA_END not fully implemented

---

## Protocol v2 Flow

```
Station A                          Station B
---------                          ---------
1. PING (1s) -------------------->
   <------------------------ PONG (1s)

2. CONNECT (DPSK) --------------->
   <------------------ CONNECT_ACK (DPSK)

3. MODE_CHANGE ------------------>  (SNR-based negotiation)
   <--------------------------- ACK

4. DATA ------------------------->  (negotiated waveform)
   <--------------------------- ACK

5. DISCONNECT ------------------->
   <--------------------------- ACK
```

---

## Development Workflow

### Before Making Changes
1. Read `docs/PROJECT_GOALS.md` and confirm the task aligns with the mission
2. Read `docs/INVARIANTS.md` for the subsystem you're touching
3. Check `docs/KNOWN_BUGS.md` for related issues
4. For agent work, use `docs/AGENT_TASK_BACKLOG.md` and one queued task file

### After Making Changes
1. Run `./build/cli_simulator --snr 15 --fading good --rate r1_4 --test 2>&1 | tee /tmp/test_output.log`
2. If you fixed a bug: Add entry to `docs/CHANGELOG.md`
3. If you discovered a bug: Add entry to `docs/KNOWN_BUGS.md`
4. If project/agent state changed materially: Update `docs/AGENT_CURRENT_STATE.md`, `docs/QUALITY_AUDIT.md`, or `docs/AGENT_TASK_BACKLOG.md` as appropriate

### Commit Message Format
```
Short description (imperative mood)

- What was changed
- Why it was changed

Fixes: BUG-XXX (if applicable)
```

---
> Source: [secup/ProjectUltra](https://github.com/secup/ProjectUltra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
