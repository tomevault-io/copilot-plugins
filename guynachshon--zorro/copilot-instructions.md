## zorro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview
Zorro is a comprehensive malware detection framework for package ecosystems with two complementary approaches:

1. **ICN (Intent Convergence Networks)**: Advanced neural architecture detecting malware through intent divergence analysis
2. **AMIL (Attention-based Multiple Instance Learning)**: Lightweight attention-based classifier optimized for CI/CD and registry scanning

## Core Architecture

### ICN Model Architecture (`icn/`)
The ICN system operates on two levels:
1. **Local Intent Estimator**: Analyzes individual code units (functions/files) for behavioral intent
2. **Global Intent Integrator**: Maintains package-level state and detects convergence patterns
3. **Dual Detection Channels**:
   - **Divergence Detection**: Identifies trojaned packages where local intents misalign with global intent
   - **Plausibility Detection**: Flags packages where global intent profile is abnormal

Key modules:
- `icn/models/icn_model.py` - Main neural network architecture with convergence loop
- `icn/training/` - Complete training pipeline with curriculum learning
- `icn/evaluation/` - Comprehensive benchmarking framework against SOTA models
- `icn/data/` - Data preparation and preprocessing pipeline
- `icn/parsing/` - Code parsing, tokenization, and AST analysis

### Evaluation Framework (`icn/evaluation/`)
Comprehensive benchmarking system comparing ICN against:
- **Security Models**: Endor Labs BERT, other HuggingFace models
- **Modern LLMs**: GPT o1, Claude 3.5, Gemini, open source models via OpenRouter
- **Traditional Baselines**: Heuristic detection, random baselines

Supports multiple evaluation modes:
- Package-level vs file-by-file analysis for LLMs
- Cross-ecosystem evaluation (npm ↔ PyPI)
- Cost/performance trade-off analysis

### AMIL Model Architecture (`amil/`)
Attention-based Multiple Instance Learning system for fast package classification:
1. **Feature Extractor**: Combines code embeddings (GraphCodeBERT) with handcrafted features (API patterns, entropy, metadata)
2. **Attention Pooling**: Multi-head attention aggregates unit representations for package-level prediction
3. **Binary Classifier**: Malicious/benign classification with confidence scores and interpretable attention weights

Key modules:
- `amil/model.py` - Core AMIL architecture with attention-based MIL pooling
- `amil/trainer.py` - 3-stage curriculum learning pipeline (balanced → augmented → realistic)
- `amil/evaluator.py` - Comprehensive evaluation (classification, localization, speed, robustness)
- `amil/feature_extractor.py` - Combined code embedding and handcrafted feature extraction
- `amil_benchmark_integration.py` - Integration with ICN evaluation framework

## Development Commands

### ICN Training
```bash
# Full ICN training pipeline with curriculum learning
python train_icn.py

# Quick structure test
python test_icn_structure.py

# Verify complete pipeline
python verify_icn_pipeline.py
```

### Benchmarking
```bash
# Quick benchmark test
python test_benchmark_framework.py

# Full benchmark study (requires OpenRouter API key)
python run_icn_benchmark.py --include-llms --include-huggingface --include-baselines

# LLM granularity comparison
python icn/evaluation/test_granularity.py
```

### Data Preparation
```bash
# Extract malicious samples from dataset
python extract_malicious_samples.py

# ICN demo on sample data
python icn_demo.py
python icn_phase2_demo.py
```

### AMIL Training & Evaluation
```bash
# AMIL demo with comprehensive examples
python amil_demo.py

# AMIL model training (3-stage curriculum)
python -m amil.trainer --config-file amil_config.json

# AMIL comprehensive evaluation  
python -m amil.evaluator --model-path checkpoints/amil_model.pth --test-data data/test_samples/

# AMIL benchmark integration test
python amil_benchmark_integration.py
```

### Package Management
```bash
# Install dependencies using uv
uv sync

# Add a new dependency  
uv add <package-name>

# Update dependencies
uv lock --upgrade
```

## Training Configuration

### Curriculum Learning (4-Stage Training)
1. **Intent Pretraining**: Local estimators learn intent classification on benign packages
2. **Convergence Training**: Global integrator learns stable convergence on benign packages  
3. **Malicious Training**: Introduce real malware, train divergence detection
4. **Robustness Training**: Adversarial examples, obfuscated code, synthetic malware

### AMIL Curriculum Learning (3-Stage Training)
1. **Stage A - Balanced Training**: 5:1 benign:malicious ratio, clean samples
2. **Stage B - Augmented Training**: Add obfuscation variants (minification, base64, string splitting)  
3. **Stage C - Realistic Training**: 10:1 ratio for production-realistic calibration

### Configuration Files
- Training: `icn/training/config.py` - model, curriculum, optimization settings
- Experiments: `icn/training/wandb_config.py` - W&B experiment tracking
- Benchmarking: `run_icn_benchmark.py` - evaluation configuration
- AMIL: `amil/config.py` - AMIL model configuration, curriculum stages, feature settings

## Environment Setup
Required environment variables:
- `GITHUB_TOKEN`: GitHub API access (for advisory scraping)
- `OPENROUTER_API_KEY`: OpenRouter API for LLM benchmarking (optional)
- `WANDB_API_KEY`: W&B experiment tracking (optional)

## Key Data Paths
- `malicious-software-packages-dataset/` - Primary malware dataset
- `data/` - Processed training data, splits, embeddings
- `checkpoints/` - Model checkpoints and saved states  
- `logs/` - Training and evaluation logs
- `test_results/` - Benchmark results and analysis

## Intent Vocabulary
ICN uses a dual intent system:
- **Fixed Intents** (~15 categories): `net.outbound`, `fs.read`, `fs.write`, `proc.spawn`, `eval`, `crypto`, `sys.env`, etc.
- **Latent Intents** (8-12 learned slots): Discovered through contrastive learning, can be promoted to fixed taxonomy

## Evaluation Metrics

### ICN Evaluation
Primary metrics for model comparison:
- **F1 Score**: Primary comparison metric
- **ROC-AUC**: Area under curve analysis  
- **Speed**: Inference time per package
- **Cost**: API costs for LLM models
- **Interpretability**: Quality of explanations and flagged units

### AMIL Success Criteria  
Production-ready targets:
- **Detection**: ROC-AUC ≥ 0.95
- **Precision**: False positive rate < 2% on benign libraries
- **Localization**: IoU ≥ 0.7 (attention weights overlap malicious units)  
- **Speed**: ≤2s inference per package
- **Robustness**: Maintained performance under obfuscation

## Package Manager Support
- **npm**: JavaScript/Node.js packages
- **PyPI**: Python packages  
- **Cargo**: Rust packages (planned)

Cross-ecosystem evaluation tests generalization between package managers.

## Important Notes
- Uses `uv` for Python dependency management (not pip directly)
- Virtual environment at `.venv/`
- ICN model training requires GPU (CUDA recommended)
- LLM benchmarking requires OpenRouter API access for cost efficiency
- All relative imports in ICN modules use `..` notation - run scripts from project root  
- Benchmark results include statistical significance testing for research publication
- AMIL can integrate with existing ICN benchmark framework for direct comparison
- Both ICN and AMIL support the same package ecosystems and data formats

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GuyNachshon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/GuyNachshon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
