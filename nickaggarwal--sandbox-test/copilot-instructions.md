## sandbox-test

> - Python virtual environment is at `./venv`

# Sandbox Benchmark Project

## Environment

- Python virtual environment is at `./venv`
- Activate before running: `source venv/bin/activate`
- All provider SDKs (daytona, e2b, blaxel, modal, runloop) and dependencies are installed in the venv

## Running Benchmarks

```bash
source venv/bin/activate
python3 run_parallel_profiled.py --benchmark fs --provider all --sandboxes 4 --max-workers 4 --stagger 1.0
```

## API Keys

Stored in `.env` file (DAYTONA_API_KEY, E2B_API_KEY, BLAXEL_API_KEY, RUNLOOP_API_KEY, GEMINI_API_KEY).

---
> Source: [nickaggarwal/sandbox-test](https://github.com/nickaggarwal/sandbox-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
