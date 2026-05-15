## sre-skills-bench

> This file provides guidance to Claude Code when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Project Overview

SRE-skills-bench is a benchmark suite for evaluating LLMs on Site Reliability Engineering tasks. It helps reliability practitioners choose the right model for IDE assistants, operational workflows, and incident response.

## Tech Stack

- **Language**: Python 3.12
- **Package Manager**: uv
- **Tool Version Manager**: mise

## Project Structure

```
.
├── src/                  # Source code
│   └── plot_benchmark.py # Visualization for benchmark results
├── scripts/              # Automation scripts
│   ├── .env.example      # Environment variables template
│   └── run-all-sre-skills-bench-tasks.sh  # Main evaluation runner
├── static/               # Images and assets
├── mise.toml             # Tool version configuration
└── pyproject.toml        # Python project configuration
```

## Development Setup

```bash
# Install tools via mise
mise trust
mise install

# Create virtual environment and install dependencies
uv venv
source .venv/bin/activate
uv pip install openbench
```

## Running Evaluations

```bash
# Set up environment
cp scripts/.env.example scripts/.env
# Edit scripts/.env with API keys

# Run all evaluations
cd scripts
./run-all-sre-skills-bench-tasks.sh
```

## Dependencies

- `openbench` - Benchmark evaluation framework
- `matplotlib` - Plotting results
- `adjusttext` - Text positioning in plots

---
> Source: [Rootly-AI-Labs/sre-skills-bench](https://github.com/Rootly-AI-Labs/sre-skills-bench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
