## mix-instruct-simulation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Mix-Instruct Concurrent Simulation Framework for GPU-based LLM routing research. The system uses vLLM to deploy multiple instruction-tuned language models on shared GPUs, simulates concurrent user requests, and monitors runtime GPU metrics to study performance under various load conditions.

## Storage Configuration

This project uses **Blue Storage** on the HPC cluster for caching models and datasets:
- Models cache: `/blue/sgao1/ji757406.ucf/hf_cache/`
- Dataset cache: `/blue/sgao1/ji757406.ucf/datasets/`

All models and the mix-instruct dataset are automatically downloaded to blue storage to avoid redundant downloads and ensure fast access during experiments.

## Architecture

The system consists of five independent modules that communicate through REST APIs and CSV logs:

1. **vLLM Servers** - Multiple model instances running on different GPUs with configurable memory allocation
2. **Load Generator** (`simulate_load.py`) - Async request simulator using asyncio + aiohttp
3. **Metrics Collector** (`collect_metrics.py`) - Prometheus endpoint polling for GPU metrics
4. **Experiment Controller** (`experiment_runner.py`) - Orchestrates experiments based on config.yaml
5. **Visualizer** (`plot_metrics.py`) - Generates plots and analysis from collected metrics

## Development Commands

### Initial Setup (First Time Only)

Before running experiments, download the dataset to blue storage:

```bash
# Option 1: Download dataset only (recommended)
python download_to_blue_storage.py --dataset-only

# Option 2: Using SLURM on HPC (recommended)
sbatch download_data.slurm
```

**Note**: Models are automatically downloaded by vLLM when servers start. They are cached to blue storage for reuse. If you want to pre-download models manually:

```bash
# Optional: Pre-download models (requires GPU)
python download_to_blue_storage.py --models-only --tensor-parallel-size 1
```

### Server Management
```bash
# Launch all vLLM servers with GPU allocation (uses blue storage automatically)
./launch_vllm_servers.sh

# Verify server health
curl http://localhost:8000/health  # Check individual servers

# Stop all servers
./stop_vllm_servers.sh
```

### Running Experiments on HPC

**Option 1: Interactive GPU Session (if already in srun/salloc)**
```bash
# Download dataset (one-time)
./download_data.sh

# Run all experiments
./run_experiments.sh
```

**Option 2: SLURM Batch Jobs**
```bash
# Submit experiment job to SLURM
sbatch run_experiments.slurm

# Check job status
squeue -u $USER

# View job output
tail -f logs/experiment-<job_id>.out
```

### Running Experiments
```bash
# Low concurrency baseline
python simulate_load.py --num_requests 10 --concurrency 2

# Medium load test
python simulate_load.py --num_requests 30 --concurrency 10

# High load / saturation test
python simulate_load.py --num_requests 100 --concurrency 30

# Full orchestrated experiment
python experiment_runner.py --config config.yaml
```

### Metrics and Visualization
```bash
# Collect metrics from running servers
python collect_metrics.py --model <model_name> --duration 300

# Generate plots from collected data
python plot_metrics.py --input metrics/<model>_metrics.csv
```

## GPU Allocation Plan

The default configuration uses 2 GPUs with the following allocation:

| GPU | Model | Port | gpu-memory-utilization |
|-----|-------|------|------------------------|
| 0 | Phi-3 Mini 3.8B | 8000 | 0.3 |
| 0 | Qwen2.5 14B | 8001 | 0.6 |
| 1 | Qwen2 3B | 8002 | 0.2 |
| 1 | Mistral 7B | 8003 | 0.4 |
| 1 | Llama-3.1 8B | 8004 | 0.3 |

## Key Metrics Monitored

- `num_requests_running` - Active requests being processed
- `num_requests_waiting` - Queued requests awaiting GPU availability
- `kv_cache_usage_perc` - KV cache utilization percentage
- `ttft` - Time to First Token
- `tpot` - Time Per Output Token
- `e2e_latency` - End-to-end request latency

## Data Flow

1. Load generator sends async requests to vLLM servers
2. Metrics collector polls `/metrics` endpoints every 2 seconds
3. Per-request metrics saved to `logs/requests_<model>.csv`
4. Server metrics saved to `metrics/<model>_metrics.csv`
5. Experiment controller synchronizes logs by timestamp
6. Visualizer produces plots in `plots/` directory

## Expected Behavior Under Load

- **Low concurrency**: No waiting queue, stable KV usage (<50%), consistent latency
- **Medium load**: `num_requests_waiting` increases, TTFT rises moderately
- **High load**: KV cache near full (~100%), significant latency spikes, queue buildup
- **Recovery**: Metrics drop to zero after load stops

## Configuration

All experiment parameters are defined in `config.yaml`:
- Model selection and GPU assignments
- Concurrency levels and request counts
- Metrics collection intervals
- Output paths for logs and plots

---
> Source: [jqxue1999/mix-instruct-simulation](https://github.com/jqxue1999/mix-instruct-simulation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
