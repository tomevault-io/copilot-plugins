## tap-detect

> > **TAP-Detect**: Temporal-Aware Perplexity Detection for AI-Generated Text

# AGENTS.md - TAP-Detect Implementation Guide

> **TAP-Detect**: Temporal-Aware Perplexity Detection for AI-Generated Text
> 
> This document provides complete developer instructions, scaffolding, and code-interface
> contracts for implementing TAP-Detect as specified in the project requirements.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Quick Start](#quick-start)
3. [Architecture](#architecture)
4. [Interface Contracts](#interface-contracts)
5. [Implementation Guide](#implementation-guide)
6. [Person A Tasks (Data Pipeline)](#person-a-tasks-data-pipeline)
7. [Person B Tasks (Model & Training)](#person-b-tasks-model--training)
8. [Review Process](#review-process)
9. [HPC Deployment](#hpc-deployment)
10. [Appendix: Research Paper Summaries](#appendix-research-paper-summaries)

---

## Project Overview

### Goal
Implement **TAP-Detect**, a novel AI-generated text detection method that addresses five critical 
limitations of traditional perplexity-based approaches by exploiting temporal dynamics in AI text 
generation.

### Key Innovation
AI models become increasingly predictable as they generate text (24-32% volatility reduction), 
while human writing maintains consistent unpredictability throughout.

### Target Performance
- **Accuracy**: 88-91%
- **Speed**: 10x faster than baseline methods
- **Minimum text length**: 150 tokens (vs 400-500 for competitors)

### Environment
- **Python**: 3.10.9
- **PyTorch**: 2.1.2+cu121
- **CUDA**: Driver 12.8, Toolkit 12.1
- **HPC**: Multi-node/multi-GPU with shared parallel filesystem

---

## Quick Start

### 1. Environment Setup

```bash
# Windows
scripts\setup_env.bat

# Linux/macOS
chmod +x scripts/setup_env.sh
./scripts/setup_env.sh

# Or manually with conda
conda env create -f environment.yml
conda activate tap-detect
```

### 2. Verify Installation

```bash
python scripts/verify_cuda.py
```

### 3. Run Tests

```bash
pytest tests/ -v
```

### 4. Train Model

```bash
# Single GPU
python training/train.py --config configs/default.yaml

# Multi-GPU (DDP)
./scripts/launch_torchrun.sh --config configs/default.yaml --gpus 4

# HPC (SLURM)
sbatch scripts/slurm_template.sh
```

---

## Architecture

### Eight Core Components

TAP-Detect implements eight interconnected components:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TAP-Detect Pipeline                             │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │  1. Key Token    │───▶│  2. Temporal     │───▶│  3. Uncertainty  │  │
│  │  Identification  │    │  Dynamics        │    │  Map Generation  │  │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘  │
│          │                       │                       │              │
│          ▼                       ▼                       ▼              │
│  ┌──────────────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │  4. Adaptive     │◀───│  5. Per-Window   │◀───│  6. Triple-      │  │
│  │  Sliding Windows │    │  Perplexity      │    │  Weighted Agg    │  │
│  └──────────────────┘    └──────────────────┘    └──────────────────┘  │
│          │                                               │              │
│          ▼                                               ▼              │
│  ┌──────────────────┐                         ┌──────────────────┐     │
│  │  7. Final Score  │◀────────────────────────│  8. Threshold    │     │
│  │  Calculation     │                         │  Classification  │     │
│  └──────────────────┘                         └──────────────────┘     │
│                                                         │              │
│                                                         ▼              │
│                                            ┌──────────────────────┐    │
│                                            │ Output: AI/Human/    │    │
│                                            │ Uncertain + Score    │    │
│                                            └──────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

| # | Component | Purpose | Key Algorithm |
|---|-----------|---------|---------------|
| 1 | Key Token ID | Select informative tokens | `top_30%(|PPL_long - PPL_short| × attention)` |
| 2 | Temporal Dynamics | Extract volatility features | Derivative dispersion, local volatility, decay ratio |
| 3 | Uncertainty Map | Measure token difficulty | `uncertainty[i] = 1 - max(P(token[i]))` |
| 4 | Adaptive Windows | Variable window sizing | Size based on local uncertainty |
| 5 | Per-Window PPL | Compute perplexity | Standard perplexity per window |
| 6 | Triple-Weighted | Aggregate with weights | Importance × Position × Uncertainty (learned) |
| 7 | Final Score | Combine signals | `w_ppl × weighted_ppl + w_temp × temporal` (learned) |
| 8 | Classification | Output decision | Thresholds: 0.3 (AI), 0.7 (Human) |

> **Note on Learnable Weights:** All combination weights (Components 2, 6, 7) are learned 
> end-to-end during training rather than being fixed hyperparameters. This eliminates 
> the need to justify specific weight choices and allows the model to adapt to different 
> datasets. The config values serve as initialization.

---

## Interface Contracts

### Contract 1: Batch Structure

All data loaders must produce batches conforming to this structure:

```python
from typing import Dict, List, Optional
import torch

class Batch(TypedDict):
    """Standard batch format for TAP-Detect pipeline."""
    
    input_ids: torch.LongTensor      # Shape: [B, L] - Token IDs
    attention_mask: torch.LongTensor  # Shape: [B, L] - 1 for real, 0 for padding
    labels: Optional[torch.LongTensor] # Shape: [B] - 0=AI, 1=Human, None for inference
    meta: List[Dict]                   # Length B - Metadata per sample
    
# Example batch:
batch = {
    "input_ids": torch.tensor([[101, 2003, ...], [101, 1996, ...]]),  # [2, 512]
    "attention_mask": torch.tensor([[1, 1, ...], [1, 1, ...]]),       # [2, 512]
    "labels": torch.tensor([0, 1]),                                    # [2]
    "meta": [
        {"source": "gpt4", "domain": "news", "length": 487},
        {"source": "human", "domain": "news", "length": 512}
    ]
}
```

### Contract 2: Model API

```python
class TAPDetectModel(nn.Module):
    """TAP-Detect model interface contract."""
    
    def forward(
        self,
        input_ids: torch.LongTensor,
        attention_mask: torch.LongTensor,
        labels: Optional[torch.LongTensor] = None,
        **kwargs
    ) -> Dict[str, torch.Tensor]:
        """
        Forward pass for TAP-Detect.
        
        Args:
            input_ids: Token IDs [B, L]
            attention_mask: Attention mask [B, L]
            labels: Ground truth labels [B], optional
            
        Returns:
            Dict containing:
                - "logits": Classification logits [B, num_classes]
                - "scores": Detection scores [B]
                - "loss": Scalar loss (only if labels provided)
                - "temporal_features": Dict of temporal features
        """
        ...
    
    def save_checkpoint(self, path: str) -> None:
        """Save model checkpoint with metadata."""
        ...
    
    @classmethod
    def load_checkpoint(
        cls, 
        path: str, 
        map_location: str = "cpu"
    ) -> "TAPDetectModel":
        """Load model from checkpoint."""
        ...
```

### Contract 3: Trainer API

```python
class Trainer:
    """Training loop interface contract."""
    
    def __init__(self, config: Dict):
        """Initialize trainer with configuration."""
        ...
    
    def fit(
        self,
        model: TAPDetectModel,
        train_dataloader: DataLoader,
        val_dataloader: Optional[DataLoader] = None,
        config: Optional[Dict] = None
    ) -> Dict[str, Any]:
        """
        Train the model.
        
        Args:
            model: TAP-Detect model instance
            train_dataloader: Training data
            val_dataloader: Validation data (optional)
            config: Override config (optional)
            
        Returns:
            Dict containing:
                - "best_metrics": Best validation metrics
                - "final_metrics": Final epoch metrics
                - "checkpoint_path": Path to best checkpoint
        """
        ...
```

### Contract 4: Data Loader API

```python
def get_dataloader(
    split: str,
    config: Dict,
    tokenizer: Optional[PreTrainedTokenizer] = None
) -> DataLoader:
    """
    Create dataloader for specified split.
    
    Args:
        split: One of "train", "val", "test"
        config: Configuration dict with data settings
        tokenizer: Tokenizer (will load default if None)
        
    Returns:
        PyTorch DataLoader yielding Batch dicts
    """
    ...
```

### Contract 5: Inference API

```python
def predict_batch(
    model: TAPDetectModel,
    batch: Batch,
    config: Optional[Dict] = None
) -> Dict[str, Any]:
    """
    Run inference on a batch.
    
    Args:
        model: Trained TAP-Detect model
        batch: Input batch
        config: Inference config (optional)
        
    Returns:
        Dict containing:
            - "predictions": List[str] - "AI-Generated", "Human-Written", "Uncertain"
            - "scores": List[float] - Detection scores
            - "confidences": List[float] - Confidence values
            - "temporal_features": List[Dict] - Per-sample features
    """
    ...
```

---

## Implementation Guide

### Directory Structure

```
TAP-Detect/
├── AGENTS.md                 # This file
├── TAP-Detect_initial.md     # Research design document
├── environment.yml           # Conda environment
├── requirements.txt          # Pip requirements
├── configs/
│   └── default.yaml          # Default configuration
├── data/
│   ├── __init__.py
│   ├── loader.py             # DataLoader factories [Person A]
│   ├── tokenizer_wrapper.py  # Tokenization [Person A]
│   ├── datasets.py           # Dataset classes [Person A]
│   ├── augmentations.py      # Data augmentation [Person A]
│   └── utils.py              # Validation utilities [Person A]
├── model/
│   ├── __init__.py
│   ├── architecture.py       # TAP-Detect model [Person B]
│   └── losses.py             # Loss functions [Person B]
├── training/
│   ├── __init__.py
│   ├── trainer.py            # Training loop [Person B]
│   ├── optim.py              # Optimizers/schedulers [Person B]
│   └── train.py              # Entry point script [Person B]
├── inference/
│   ├── __init__.py
│   └── serve.py              # Inference engine [Person B]
├── tests/
│   ├── __init__.py
│   ├── test_data_pipeline.py # Data tests [Person A]
│   ├── test_forward_backward.py # Model tests [Person B]
│   └── test_contracts.py     # Integration tests [Both]
├── scripts/
│   ├── setup_env.sh          # Linux setup
│   ├── setup_env.bat         # Windows setup
│   ├── verify_cuda.py        # CUDA validation
│   ├── launch_torchrun.sh    # DDP launcher
│   └── slurm_template.sh     # HPC job template
├── docs/
│   ├── Codex-initial.xml    # Original prompt
│   ├── TAP-Detect_initial.md # Research design
│   ├── paper_summaries.md    # Research paper analysis
│   └── reviews/              # Design review logs
│       └── review_1.md
└── .github/
    └── workflows/
        └── ci.yaml           # CI/CD pipeline
```

---

## Person A Tasks (Data Pipeline)

### Deliverables

| File | Description | Priority |
|------|-------------|----------|
| `data/loader.py` | DataLoader with dynamic batching, bucketing, prefetch | High |
| `data/tokenizer_wrapper.py` | Consistent tokenization wrapper | High |
| `data/datasets.py` | PyTorch Dataset implementations | High |
| `data/augmentations.py` | Data augmentation strategies | Medium |
| `data/utils.py` | Validation and utility functions | Medium |
| `tests/test_data_pipeline.py` | Comprehensive data tests | High |
| `DATA_README.md` | Data module documentation | Low |

### Detailed Requirements

#### 1. TokenizerWrapper (`data/tokenizer_wrapper.py`)

```python
class TokenizerWrapper:
    """
    Requirements:
    - Wrap transformers tokenizer with consistent options
    - Support: padding, truncation, special tokens
    - Max length: configurable (default 512)
    - Return attention masks
    - Handle batched and single inputs
    """
```

#### 2. TAPDetectDataset (`data/datasets.py`)

```python
class TAPDetectDataset(Dataset):
    """
    Requirements:
    - Load from multiple data sources (JSON, CSV, HuggingFace)
    - Validate inputs: check for empty, corrupt, too-long sequences
    - Return standardized Batch format
    - Support lazy loading for large datasets
    - Include metadata (source, domain, length)
    """
```

#### 3. DataLoader Factory (`data/loader.py`)

```python
def get_dataloader(split: str, config: Dict) -> DataLoader:
    """
    Requirements:
    - Dynamic batching by sequence length (bucketing)
    - Configurable: batch_size, num_workers, pin_memory
    - Prefetch support for HPC
    - Reproducible with seed control
    - Handle distributed sampling for DDP
    """
```

#### 4. Data Validation (`data/utils.py`)

```python
def validate_dataset(path: str) -> ValidationReport:
    """
    Requirements:
    - Check file integrity (checksums)
    - Validate schema (required fields)
    - Detect: empty files, corrupt lines, unexpected labels
    - Report: statistics, warnings, errors
    """
```

### Test Requirements

```python
# tests/test_data_pipeline.py

def test_batch_structure():
    """Verify batch conforms to Contract 1."""
    
def test_tokenizer_consistency():
    """Verify tokenization is reproducible."""
    
def test_dataloader_shapes():
    """Verify output shapes match spec."""
    
def test_empty_input_handling():
    """Verify graceful handling of edge cases."""
    
def test_distributed_sampling():
    """Verify correct sampling in DDP mode."""
```

---

## Person B Tasks (Model & Training)

### Deliverables

| File | Description | Priority |
|------|-------------|----------|
| `model/architecture.py` | TAP-Detect model with 8 components | High |
| `model/losses.py` | Detection loss functions | High |
| `training/trainer.py` | Training loop with AMP, DDP, checkpointing | High |
| `training/optim.py` | Optimizer/scheduler configuration | Medium |
| `training/train.py` | Training entry point script | High |
| `inference/serve.py` | Inference with rate limiting | Medium |
| `tests/test_forward_backward.py` | Model tests | High |
| `MODEL_README.md` | Model documentation | Low |

### Detailed Requirements

#### 1. TAPDetectModel (`model/architecture.py`)

```python
class TAPDetectModel(nn.Module):
    """
    Implement all 8 components:
    
    1. Key Token Identification
       - Long/short context perplexity comparison
       - Attention weight extraction
       - Top 30% selection
    
    2. Temporal Dynamics Extraction
       - Derivative dispersion: std(diff(log_probs))
       - Local volatility: rolling_std(window=5)
       - Volatility decay ratio
    
    3. Uncertainty Map Generation
       - uncertainty[i] = 1 - max(P(token[i]))
    
    4. Adaptive Sliding Windows
       - High uncertainty (>0.7): window=32, stride=8
       - Medium (0.3-0.7): window=64, stride=16
       - Low (<0.3): window=128, stride=32
    
    5. Per-Window Perplexity
       - Standard perplexity calculation
    
    6. Triple-Weighted Aggregation (LEARNABLE WEIGHTS)
       - W_importance: content vs function word weight (learned)
       - W_position: 1 + factor × (pos / length) (factor learned)
       - W_uncertainty: from uncertainty map
    
    7. Final Score Calculation (LEARNABLE WEIGHTS)
       - w_ppl × weighted_ppl + w_temp × temporal_adjustment
       - Weights w_ppl, w_temp learned end-to-end
    
    8. Threshold Classification
       - < 0.3: AI-Generated
       - > 0.7: Human-Written
       - 0.3-0.7: Uncertain
    """
```

#### 2. Training Loop (`training/trainer.py`)

```python
class Trainer:
    """
    Requirements:
    - Automatic Mixed Precision (torch.cuda.amp)
    - Gradient accumulation for large effective batch
    - Distributed Data Parallel (DDP) support
    - Checkpoint save/resume with atomic writes
    - Learning rate scheduling (warmup + decay)
    - Early stopping on validation metric
    - Graceful SIGTERM/SIGINT handling
    - Logging: tensorboard, wandb (optional)
    - Memory-aware batch size reduction on OOM
    """
```

#### 3. Inference Engine (`inference/serve.py`)

```python
class InferenceEngine:
    """
    Requirements:
    - Batch inference with configurable batch size
    - Token-bucket rate limiter
    - Configurable timeout per request
    - Graceful degradation under load
    - Health check endpoint
    - Metrics: latency, throughput, queue depth
    """
```

### Test Requirements

```python
# tests/test_forward_backward.py

def test_forward_output_shapes():
    """Verify model output shapes match Contract 2."""
    
def test_backward_gradients():
    """Verify gradients flow correctly."""
    
def test_checkpoint_save_load():
    """Verify checkpoint round-trip."""
    
def test_cpu_smoke_test():
    """Verify model runs on CPU (no GPU required)."""
    
def test_temporal_features():
    """Verify temporal feature extraction."""
```

---

## Review Process

### Design Review #1: Interface Review

**Checklist:**
- [ ] All research-paper-derived components mapped to modules
- [ ] Batch format and model API contracts documented
- [ ] Config schema complete: training, optimizer, data paths, HPC options
- [ ] Person A and Person B deliverables clearly assigned

**Action:** Record decisions in `docs/reviews/review_1.md`

### Design Review #2: Implementation Smoke Review

**Checklist:**
- [ ] Stubs and unit tests pass on CPU
- [ ] CI pipeline runs static checks (black, flake8, mypy)
- [ ] Checkpointing and resume logic testable without GPUs

**Action:** Fix integration failures, extend edge case tests

### Design Review #3: HPC Pre-deploy Review

**Checklist:**
- [ ] Distributed/torchrun launch scripts tested on 1 GPU node
- [ ] Memory/multiprocessing settings tuned
- [ ] Profiling and OOM mitigation plan documented
- [ ] Rate limiting and inference protections ready

**Action:** Authorize production runs

---

## HPC Deployment

### Environment Setup on HPC

```bash
# Load modules (adjust for your HPC)
module load cuda/12.1 cudnn/8.9.0

# Create environment
./scripts/setup_env.sh

# Verify
python scripts/verify_cuda.py
```

### Single-Node Multi-GPU

```bash
./scripts/launch_torchrun.sh --gpus 4 --config configs/default.yaml
```

### Multi-Node Training

```bash
# On node 0
MASTER_ADDR=node0 NODE_RANK=0 NUM_NODES=2 ./scripts/launch_torchrun.sh

# On node 1
MASTER_ADDR=node0 NODE_RANK=1 NUM_NODES=2 ./scripts/launch_torchrun.sh
```

### SLURM Submission

```bash
# Edit scripts/slurm_template.sh for your cluster
sbatch scripts/slurm_template.sh
```

### Memory Optimization

```yaml
# configs/default.yaml
training:
  gradient_accumulation_steps: 4  # Increase for larger effective batch
  gradient_checkpointing: true     # Trade compute for memory
  mixed_precision: true            # FP16 for 2x memory reduction
  max_memory_per_gpu: 24GB         # Auto batch reduction if exceeded
```

---

## Appendix: Research Paper Summaries

### 1. DetectGPT (ICML 2023)

**Core Algorithm:** Probability curvature detection
```
d = log p(x) - E[log p(x̃)]
```
where x̃ = perturbed versions using T5

**Key Insights:**
- AI text occupies negative curvature regions of log probability
- Perturbation-based approach requires ~100 model calls
- AUROC: 0.95 on GPT-NeoX fake news

**TAP-Detect Usage:** Baseline comparison; informs perplexity computation

### 2. Fast-DetectGPT (ICLR 2024)

**Core Algorithm:** Conditional probability curvature
- Replaces perturbation with sampling step
- 340x faster than DetectGPT
- 75% relative improvement in detection

**TAP-Detect Usage:** Validates our adaptive windowing speedup approach

### 3. DetectLLM

**Core Algorithm:** Log-rank information
- LRR: Log Rank Ratio (fast)
- NPR: Normalized Perturbed Rank (accurate)

**Key Insights:**
- 3.9 AUROC improvement over prior work
- Fewer perturbations needed

**TAP-Detect Usage:** Alternative scoring for ablation studies

### 4. LongPPL (ACL 2024)

**Core Algorithm:** Long-short context comparison
```python
key_tokens = where(|PPL_long - PPL_short| is large)
```

**Key Insights:**
- Not all tokens equally informative
- Long context reveals key discriminative tokens

**TAP-Detect Usage:** Directly used in Component 1 (key token identification)

---

## Configuration Reference

See `configs/default.yaml` for full configuration schema.

---

*Last updated: 2026-01-29*
*Version: 1.0.0*

---
> Source: [k25kar/TAP-Detect](https://github.com/k25kar/TAP-Detect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
