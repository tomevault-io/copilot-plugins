## smiles-transformer

> **SMILES-Transformer** is a machine learning framework for training transformer models on chemical reactions and molecules using SMILES notation and Condensed Graph of Reaction (CGR) representations.

# CLAUDE.MD - Project Context for AI Assistant

## Project Overview

**SMILES-Transformer** is a machine learning framework for training transformer models on chemical reactions and molecules using SMILES notation and Condensed Graph of Reaction (CGR) representations.

**Author**: Giustino Sulpizio
**Version**: 0.1
**Python**: 3.13

## Project Purpose

This library enables:
- Pre-training transformer models on chemical reaction data
- Fine-tuning models for regression/classification tasks on molecular properties
- Predicting chemical properties using trained models
- Cross-validation experiments for model evaluation
- Working with SMILES, CGR, and SR-SMILES representations

## Architecture Overview

### Core Components

1. **Dataset Management** (`smiles_transformer/dataset/`)
   - `basedatasetfactory.py` - Base factory for dataset creation
   - `pretrainingdatasetfactory.py` - Handles pre-training data
   - `regressiondatasetfactory.py` - Regression task datasets
   - `classificationdatasetfactory.py` - Classification task datasets
   - `preprocesseddatasetfactory.py` - Pre-processed data handling

2. **Model Definitions** (`smiles_transformer/model/`)
   - `basemodelfactory.py` - Base model factory
   - `pretrainingmodelfactory.py` - Pre-training model creation
   - `finetuningmodelfactory.py` - Fine-tuning model setup
   - `translationmodelfactory.py` - Translation models
   - `featureaugmentedclassificationbert.py` - Feature-augmented BERT for classification

3. **Preprocessing Pipeline** (`smiles_transformer/preprocessing/`)
   - `basedataloader.py` - Base data loading
   - `ontheflydataloader.py` - Dynamic data loading
   - `premadedataloader.py` - Pre-made dataset loading
   - **Transform Operations** (`transform/`):
     - CGR conversions (`smilestocgrtransform.py`, `cgrtostrtransform.py`)
     - Atom mapping (`rxnmaptransform.py`, `removeatommappingtransform.py`)
     - Augmentation (`augmentsmilestransform.py`)
     - Filtering (`contentfilter.py`, `mappingconfidencefilter.py`)
     - Descriptor handling (`atomdescriptorinserter.py`, `moleculardescriptorinserter.py`)

4. **Tokenizers** (`smiles_transformer/tokenizer/`)
   - `individualtokenizer.py` - Character-level tokenization
   - `bpetokenizer.py` - Byte-pair encoding tokenizer
   - `describedtokenizer.py` - Descriptor-aware tokenizer
   - `singletokenizer.py` - Single token handling

5. **Training** (`smiles_transformer/trainer/`)
   - `basetrainerfactory.py` - Base trainer factory
   - `pretrainingtrainerfactory.py` - Pre-training setup
   - `finetuningtrainerfactory.py` - Fine-tuning setup

6. **Evaluation** (`smiles_transformer/evaluation/`)
   - `evaluatortemplate.py` - Base evaluator
   - `regressionevaluator.py` - Regression metrics
   - `classificationevaluator.py` - Classification metrics

7. **Data Splitting** (`smiles_transformer/splitter/`)
   - `sklearnsplitter.py` - sklearn-based splitting
   - `cvlabeledsplitter.py` - Cross-validation splitting
   - `augmentcvlabeledsplitter.py` - Augmented CV splitting

## Key Dependencies

- **PyTorch** (>=2.6.0) - Deep learning framework
- **Transformers** (>=4.49.0) - Hugging Face transformers
- **RDKit** (>=2024.9.5) - Chemistry toolkit
- **CGRTools** (>=4.1.35) - CGR handling
- **WandB** (>=0.19.7) - Experiment tracking
- **pandas**, **datasets**, **scikit-learn** - Data handling
- Custom forks: `rxnmapper`, `sr-smiles`

## Configuration System

Configuration is managed via YAML files in `configurations/`. Key settings:

- **Paths**: `path_to_outputs`, `path_to_vocab_folder`, `dataset_folder`
- **Dataset**: `dataset_name`, `column_name`, `label_column_name`
- **Preprocessing**: `mapping`, `convert_to_cgr`
- **Model**: `model_mode` (pretraining/regression/classification)
- **Training**: `run_name`, `eval_size`

See `configurations/configurations.md` for complete documentation.

## Typical Workflows

### 1. Pre-training a Model
```bash
# Set model_mode to 'pretraining' in config.yaml
python -m smiles_transformer.main "configurations/"
```

### 2. Fine-tuning for Regression/Classification
```bash
# Set model_mode to 'regression' or 'classification'
# Set run_name to pre-training run name
python -m smiles_transformer.main "configurations/"
```

### 3. Cross-Validation
```bash
python -m smiles_transformer.launch_cv "configurations/"
```

### 4. Making Predictions
```bash
python -m smiles_transformer.predict "configurations/" "output.csv"
```

## Entry Points

- `main.py` - Main training script
- `launch_cv.py` - Cross-validation launcher
- `launch_sweep.py` - Hyperparameter sweeps
- `launch_loop_from_file.py` - Batch experiments
- `predict.py` - Inference script

## Important Notes for Development

1. **Data Format**: CSV files with reaction/molecule SMILES in specified columns
2. **WandB Integration**: Requires WandB account and local login
3. **Eval Size**: Keep `eval_size` < 1000 to avoid performance issues
4. **Atom Mapping**: Set `mapping: "remove"` for atom-mapped SMILES
5. **Installation**: May require C toolchain (build-essential) for CGRTools

## File Organization

```
smiles_transformer/
├── dataset/          # Dataset factories
├── model/            # Model definitions
├── preprocessing/    # Data loading and transforms
├── tokenizer/        # Tokenization strategies
├── trainer/          # Training logic
├── evaluation/       # Evaluation metrics
├── splitter/         # Data splitting
├── modelization/     # Metric computation
├── utils/            # Helper functions
└── visualization/    # Plotting utilities
```

## Testing and Validation

- Models saved to `path_to_outputs`
- Vocabulary files in `path_to_vocab_folder`
- WandB tracks all experiments
- Evaluation metrics logged during training

## Common Issues

1. **CGRTools Installation**: Requires C compiler (build-essential on Linux)
2. **Memory**: Large eval sets cause performance issues
3. **Mapping**: Ensure correct mapping settings for your SMILES format
4. **Paths**: All paths in config must be absolute or relative to execution directory

## Environment

- Recommended: Conda environment with Python 3.13
- Install: `pip install .` from repository root
- Container: Singularity definition available (`singularity.def`)

## Git Configuration

- Main branch: `main`
- Current worktree: `serene-bose`
- GitHub authentication configured for containers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/digital-chemistry-laboratory)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/digital-chemistry-laboratory)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
