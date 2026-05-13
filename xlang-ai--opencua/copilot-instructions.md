## opencua

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenCUA is an open-source framework for scaling computer-use agent data and foundation models. It consists of:
- **AgentNet**: Large-scale dataset (22.6K+ tasks) across Windows, macOS, and Ubuntu
- **AgentNetTool**: Cross-platform GUI recording and annotation tool (Git submodule in `tool/`)
- **OpenCUA Models**: End-to-end foundation models (7B, 32B, 72B variants) based on Qwen2.5-VL

## Commands

### Environment Setup
```bash
conda create -n opencua python=3.10
conda activate opencua

# Model inference dependencies
pip install -r model/requirement.txt

# Data processing dependencies
pip install -r data/data-process/requirements.txt

# Headless server support (for video processing)
sudo apt-get install -y xvfb
```

### Model Inference

**HuggingFace**:
```bash
cd model/inference
python huggingface_inference.py
```

**vLLM Server** (recommended for production):
```bash
CUDA_VISIBLE_DEVICES=1,2,3,4 vllm serve /path/to/OpenCUA-32B \
  --trust-remote-code \
  --tensor-parallel-size 4 \
  --served-model-name opencua-32b \
  --host 0.0.0.0 \
  --port 8000
```

### Data Processing Pipeline

**Stage 1: Extract raw trajectories** from video/events:
```bash
cd data/data-process
xvfb-run -a ./scripts/extract_raw.sh -1    # all samples
xvfb-run -a ./scripts/extract_raw.sh 10    # first 10 samples
```

**Stage 2: Standardize** to unified schema:
```bash
xvfb-run -a ./scripts/raw_to_standardized.sh -1
```

**Stage 3: Generate CoT** (requires API_KEY env var):
```bash
cd data/cot-generate
export API_KEY="your-api-key"
python gen_cot.py \
  --traj_path ./gen_cot_example/raw_example.jsonl \
  --image_folder ./gen_cot_example/images \
  --output_dir ./gen_cot_example/output/tasks \
  --model claude-3-7-sonnet-20250219
```

### Visualization
```bash
cd data/vis
streamlit run app.py -- --traj_file <TRAJ_JSONL> --meta_file <META_JSONL> --image_dir <IMAGES>
```

### AgentNetBench Evaluation
```bash
cd evaluation/agentnetbench
pip install "openai>=1.0.0" pillow editdistance

python run.py \
  --data single_data \
  --image_dir single_data/images \
  --output output \
  --model qwen2.5-vl-7b \
  --base_url http://YOUR_SERVER/v1 \
  --api_key YOUR_KEY \
  --num_cores 10

# Re-evaluate without model calls
python reeval.py --input_dir output/<eval_dir>
```

## Architecture

### Data Pipeline Flow
```
Raw demos (video + events) → data-process/extract_raw.py → raw_trajs/
                           → data-process/raw_to_standardized.py → standardized/
                           → cot-generate/gen_cot.py → enriched CoT data
```

### Key Components

**Data Processing** (`data/data-process/src/`):
- `extract_raw.py`: Extracts frames and events from raw episode folders
- `raw_to_standardized.py`: Converts to unified `Trajectory` schema
- `schema/action.py`: Comprehensive Pydantic action types (click, write, scroll, hotkey, etc.)
- `schema/trajectory.py`: Trajectory data structures with validation

**CoT Generation** (`data/cot-generate/`):
- `gen_cot.py`: Main pipeline orchestrator
- `module/reflector.py`: Error identification and reflection reasoning
- `module/generator.py`: Structured CoT generation (observation, thought, action)
- `module/evaluator.py`: Trajectory scoring (alignment, efficiency, difficulty)

**Evaluation** (`evaluation/agentnetbench/`):
- `run.py`: Main evaluation orchestrator with async OpenAI client
- `eval.py`: Action scoring logic (coordinate threshold: 0.01√2, text similarity: 0.8)
- `agent/`: Agent implementations (Qwen2.5-VL, Aguvis, OpenCUA)

**Model Inference** (`model/inference/`):
- `huggingface_inference.py`: HuggingFace model loading with base64 image encoding

### Coordinate System

All coordinates are normalized (0-1 range). The evaluation accepts clicks/moves inside bounding boxes when metadata is available. OpenCUA models output absolute coordinates post smart-resize.

### Model Modifications from Qwen2.5-VL

- M-RoPE replaced with 1D RoPE
- Custom tokenizer and chat template (Kimi-VL aligned)
- vLLM supported via `--trust-remote-code` flag

## Data Formats

**Trajectory JSON** (input to evaluation):
```json
{
  "task_id": "unique_id",
  "high_level_task_description": "...",
  "steps": [
    {
      "image": "screenshot.png",
      "ground_truth_actions": [
        { "type": "click", "params": { "position": {"x": 0.42, "y": 0.63} }, "metadata": {"bboxes": []} }
      ]
    }
  ]
}
```

**CoT Output** (enriched trajectory):
```json
{
  "task_id": "...",
  "traj": [
    {
      "image": "...",
      "value": {
        "code": "pyautogui.click(x=0.5, y=0.3)",
        "observation": "...",
        "thought": "...",
        "action": "...",
        "last_step_correct": true,
        "reflection": "..."
      }
    }
  ],
  "alignment_score": 8,
  "efficiency_score": 7
}
```

## Directory Conventions

- `datasets/raw/`: Raw episode folders (video + metadata + events)
- `datasets/raw_trajs/`: Extracted trajectory JSONs
- `datasets/standardized/`: Unified schema output
- Scripts use `python -m src.*` pattern for module invocation

---
> Source: [xlang-ai/OpenCUA](https://github.com/xlang-ai/OpenCUA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
