## asterisk-wb-checker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

HD voice (G.722 / AMR-WB) path tester for Asterisk PBX. Sends a
dual-tone probe (1 kHz + 6 kHz, 16 kHz sample rate) through a call
path, records the far end, and decides HD vs narrowband from the
FFT energy ratio between the two bands (threshold -20 dB). The
6 kHz component dies if any hop transcodes to a narrowband codec -
that is the whole detection principle. Do not "fix" anything toward
8 kHz audio; the analyzer intentionally rejects recordings that
cannot carry 6 kHz.

## Commands

- Install deps: `uv sync`
- Run tests: `uv run pytest`
- Single test: `uv run pytest tests/test_analyze.py::test_clean_path_is_hd`
- Lint: `uv run ruff check src tests` and `uv run pylint src/wb_checker`
- CLI: `uv run wb-checker {generate|analyze|call|run}`

## Architecture

Pipeline: tonegen -> Asterisk plays/records -> analyze. Python side
never touches RTP; Asterisk does all telephony via the dialplan in
`asterisk/extensions_hdtest.conf` (context `hd-test`: `play` emits
probe, `record` captures to /var/spool/asterisk/hdtest as .wav16,
`loopback` self-tests without a trunk).

- `src/wb_checker/tonegen.py` - probe synthesis, writes .sln16
  (raw s16le, Asterisk-native) and .wav
- `src/wb_checker/analyze.py` - reads .wav/.sln16, silence-trims,
  windowed rfft, band energies, `Verdict` dataclass
- `src/wb_checker/originate.py` - minimal blocking AMI client
  (stdlib socket, no deps), Originate action only
- `src/wb_checker/cli.py` - argparse subcommands; `run` = call +
  poll rec dir for new .wav + analyze; exit codes 0 HD / 1
  narrowband / 2 error (cron contract - keep stable)

Constants shared across modules (SAMPLE_RATE, LOW_HZ, HIGH_HZ,
HD_THRESHOLD_DB) live in tonegen.py/analyze.py - keep tests and
tonegen consistent with them if changed.

## Constraints

- Line length 79 (ruff + pylint configured in pyproject.toml)
- ASCII only in code, comments, logs, commit messages
- Only third-party dep is numpy; AMI client stays stdlib-only
- Tests simulate narrowband transcoding by FFT lowpass at 3.4 kHz
  (tests/test_analyze.py) - no Asterisk needed to run the suite

---
> Source: [dfgg12/asterisk-wb-checker](https://github.com/dfgg12/asterisk-wb-checker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
