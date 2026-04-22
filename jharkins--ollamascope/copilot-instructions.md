## ollamascope

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ollamascope is a bash-based telemetry wrapper for `ollama run` that captures per-run metrics including LLM performance, GPU utilization, and system resources. The entire project is a single self-contained bash script with no build process.

## Core Components

- **ollamascope.sh**: Main script (210 lines) that orchestrates metric collection
  - Uses background processes to sample nvidia-smi and dstat
  - Runs ollama with --verbose flag to capture detailed metrics
  - Parses ollama footer output to extract token counts and rates
  - Outputs organized run artifacts to `runs/<run_id>/` directories

## Commands

```bash
# Make script executable
chmod +x ollamascope.sh

# Basic run with inline prompt
./ollamascope.sh --model <model:tag> --prompt "Your prompt here"

# Run with prompt from file
./ollamascope.sh --model <model:tag> --prompt-file path/to/prompt.txt

# Run with custom sampling interval (default 1s)
./ollamascope.sh --model <model:tag> --prompt "..." --interval 0.5

# Run without GPU metrics (CPU-only systems)
./ollamascope.sh --model <model:tag> --prompt "..." --no-gpu

# Tag runs for comparison (e.g., cold vs warm)
./ollamascope.sh --model <model:tag> --prompt "..." --tag warm-cache
```

## Output Structure

Each run creates a timestamped directory under `runs/`:
- **run.log**: Raw ollama output with model response and footer metrics
- **gpu_metrics.csv**: nvidia-smi samples (utilization, memory, temp, power)
- **sys_metrics.csv**: dstat samples (CPU, RAM, load averages)
- **prompt.txt**: Exact prompt used
- **meta.txt**: System snapshot (ollama models, GPU info, etc.)
- **summary.json**: Parsed metrics (durations, token counts, rates)

## Key Implementation Details

- Footer parsing (lines 27-44): Extracts metrics using awk, strips units, handles spacing variations
- Background process management: Starts nvidia-smi and dstat samplers before ollama run, kills them after
- Run ID generation: ISO timestamp + hostname + model name + optional tag
- Error handling: Uses `set -euo pipefail`, trap for cleanup, dependency checks
- JSON generation: Prefers jq if available, falls back to manual JSON construction

## Dependencies

Required:
- bash
- ollama
- dstat

Optional:
- nvidia-smi (for GPU metrics, auto-detected)
- jq (for prettier JSON output)

## Development Notes

- The script uses POSIX-compatible date commands where possible
- All file paths are relative to the script's working directory
- The --verbose flag on ollama is critical for metric extraction
- Background processes are managed via PID tracking and trap cleanup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jharkins) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
