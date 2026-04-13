## task1-1-sprsound

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a respiratory sound classification research project that **successfully surpassed the IEEE BioCAS 2022 Grand Challenge results** on the SPRSound pediatric dataset. **OBJECTIVE COMPLETED**: Achieved **89.58%** on **Task 1-1** (binary classification: Normal vs Adventitious respiratory sounds), exceeding the target of **89.26%** by **+0.32%**.

## Key Development Commands

### Data Pipeline Execution (COMPLETED)
```python
import funciones as fc

# 1. Organize raw dataset structure ✓
fc.organizar_dataset()

# 2. Clean data (remove "Poor Quality" records) ✓
fc.clean_data(config.TRAIN_WAV_PATH, config.TRAIN_JSON_PATH, 
              config.TRAIN_WAV_CLEAN_PATH, config.TRAIN_JSON_CLEAN_PATH)
fc.clean_data(config.VAL_WAV_PATH, config.VAL_JSON_PATH, 
              config.VAL_WAV_CLEAN_PATH, config.VAL_JSON_CLEAN_PATH)
fc.clean_data(config.TEST_WAV_PATH, config.TEST_JSON_PATH, 
              config.TEST_WAV_CLEAN_PATH, config.TEST_JSON_CLEAN_PATH)

# 3. Segment audio into individual events ✓
fc.segment_data(config.TRAIN_WAV_CLEAN_PATH, config.TRAIN_JSON_CLEAN_PATH, config.TRAIN_SEGMENTED_PATH)
fc.segment_data(config.VAL_WAV_CLEAN_PATH, config.VAL_JSON_CLEAN_PATH, config.VAL_SEGMENTED_PATH)
fc.segment_data(config.TEST_WAV_CLEAN_PATH, config.TEST_JSON_CLEAN_PATH, config.TEST_SEGMENTED_PATH)

# 4. Resample and normalize audio to 16kHz standard ✓
fc.resample_and_normalize(os.path.join(config.TRAIN_SEGMENTED_PATH, "wav"))
fc.resample_and_normalize(os.path.join(config.VAL_SEGMENTED_PATH, "wav"))
fc.resample_and_normalize(os.path.join(config.TEST_SEGMENTED_PATH, "wav"))

# 5. Validate processed dataset integrity ✓
fc.validate_segmented_dataset(config.TRAIN_SEGMENTED_PATH)
fc.validate_segmented_dataset(config.VAL_SEGMENTED_PATH)
fc.validate_segmented_dataset(config.TEST_SEGMENTED_PATH)

# 6. Pad/trim audio to standard duration (2s @ 16kHz) ✓
fc.pad_or_trim_audio(config.TRAIN_SEGMENTED_PATH, config.TRAIN_TRIMMED_PATH, target_duration_sec=2.0, target_sr=16000)
fc.pad_or_trim_audio(config.VAL_SEGMENTED_PATH, config.VAL_TRIMMED_PATH, target_duration_sec=2.0, target_sr=16000)
fc.pad_or_trim_audio(config.TEST_SEGMENTED_PATH, config.TEST_TRIMMED_PATH, target_duration_sec=2.0, target_sr=16000)

# 7. Extract MFCC features ✓
fc.generate_mfcc_features(config.TRAIN_TRIMMED_PATH, config.TRAIN_MFCC_TRIMMED_PATH, sr=16000, n_mfcc=13)
fc.generate_mfcc_features(config.VAL_TRIMMED_PATH, config.VAL_MFCC_TRIMMED_PATH, sr=16000, n_mfcc=13)
fc.generate_mfcc_features(config.TEST_TRIMMED_PATH, config.TEST_MFCC_TRIMMED_PATH, sr=16000, n_mfcc=13)

# 8. Extract Log-Mel features ✓
fc.generate_logmel_features(config.TRAIN_TRIMMED_PATH, config.TRAIN_LOGMEL_TRIMMED_PATH, sr=16000, n_mels=64)
fc.generate_logmel_features(config.VAL_TRIMMED_PATH, config.VAL_LOGMEL_TRIMMED_PATH, sr=16000, n_mels=64)
fc.generate_logmel_features(config.TEST_TRIMMED_PATH, config.TEST_LOGMEL_TRIMMED_PATH, sr=16000, n_mels=64)

# 9. Extract STFT features (NEW - ECE UoIT approach) ✓
fc.generate_stft_features(config.TRAIN_TRIMMED_PATH, config.TRAIN_STFT_TRIMMED_PATH, sr=16000, n_fft=1024, hop_length=512)
fc.generate_stft_features(config.VAL_TRIMMED_PATH, config.VAL_STFT_TRIMMED_PATH, sr=16000, n_fft=1024, hop_length=512)
fc.generate_stft_features(config.TEST_TRIMMED_PATH, config.TEST_STFT_TRIMMED_PATH, sr=16000, n_fft=1024, hop_length=512)
```

### Model Training Commands (ALL COMPLETED)
```python
# 1. Classical models with MFCC features ✅
clf = fc.train_and_validate_classical_models_from_mfcc(
    train_csv="data/processed/trimmed/train/features/mfcc/labels.csv",
    val_csv="data/processed/trimmed/validation/features/mfcc/labels.csv"
)
# Results: RF=84.36%, SVM=82.87%, LR=78.39%

# 2. ResNet18 + LogMel features ✅
train_logmel_dataset = fc.LogMelDataset("data/processed/trimmed/train/features/logmel/labels.csv")
val_logmel_dataset = fc.LogMelDataset("data/processed/trimmed/validation/features/logmel/labels.csv")
train_logmel_loader = DataLoader(train_logmel_dataset, batch_size=32, shuffle=True)
val_logmel_loader = DataLoader(val_logmel_dataset, batch_size=32, shuffle=False)

resnet18_logmel_model = fc.create_resnet18_logmel_model(num_classes=2, pretrained=True)
class_weights = fc.calculate_class_weights(config.TRAIN_STFT_TRIMMED_PATH + "/labels.csv")

resnet18_logmel_trained = fc.train_resnet18_logmel(
    model=resnet18_logmel_model,
    train_loader=train_logmel_loader,
    val_loader=val_logmel_loader,
    num_epochs=25,
    learning_rate=0.001,
    use_focal_loss=True,
    class_weights=class_weights
)
# Final Score: 78.56% (SE: 97.94%, SP: 62.98%)

# 3. ResNet18 + STFT ✅
train_stft_dataset = fc.STFTDataset(config.TRAIN_STFT_TRIMMED_PATH + "/labels.csv")
val_stft_dataset = fc.STFTDataset(config.VAL_STFT_TRIMMED_PATH + "/labels.csv")

train_stft_loader = DataLoader(train_stft_dataset, batch_size=16, shuffle=True)
val_stft_loader = DataLoader(val_stft_dataset, batch_size=16, shuffle=False)

resnet18_stft_model = fc.ResNet18STFT(num_classes=2, pretrained=True)
resnet18_stft_trained = fc.train_resnet18_stft(
    model=resnet18_stft_model,
    train_loader=train_stft_loader,
    val_loader=val_stft_loader,
    num_epochs=25,
    learning_rate=0.001,
    use_focal_loss=True,
    class_weights=class_weights
)
# Final Score: 87.68% (SE: 90.75%, SP: 84.71%)

# 4. ENSEMBLE SPRSound - FINAL WINNER ✅
from load_trained_models import SPRSoundEnsemble
ensemble = SPRSoundEnsemble(models_dir="models")
ensemble.load_all_models()

# Optimized weights: RF+MFCC (80%) + ResNet18+STFT (10%) + ResNet18+LogMel (10%)
# FINAL RESULT: 89.58% Final Score - OBJECTIVE COMPLETED!
```

### Development History & Implementation
- **Primary development**: `test.ipynb` - Complete experimental pipeline
- **Data pipeline**: ✅ All functions implemented and executed (9,089 segmented samples)
- **Feature extraction**: ✅ MFCC, Log-Mel, and STFT features extracted
- **Classical ML**: ✅ RF, SVM, LR baselines (84.36% best with RF+MFCC)
- **Deep Learning**: ✅ ResNet18+STFT (87.68%) and ResNet18+LogMel (78.56%) 
- **Ensemble System**: ✅ Production-ready ensemble achieving **89.58%** final score
- **Model Persistence**: ✅ All models saved with metadata and loading utilities

## Code Architecture

### Complete Modular Pipeline Structure
```
config.py           # Central path configuration
funciones/          # Data processing modules
├── organizar_dataset.py      # Raw data → organized splits ✅
├── clean_data.py             # Remove "Poor Quality" labels ✅
├── segment_data.py           # Extract individual events from records ✅
├── resample_and_normalize.py # Audio preprocessing (16kHz, normalization) ✅
├── pad_or_trim_audio.py      # Audio duration standardization (2s) ✅
├── generate_mfcc_features.py # MFCC feature extraction ✅
├── generate_logmel_features.py # Log-Mel spectrogram features ✅
├── generate_stft_features.py # STFT feature extraction ✅
├── resnet18_stft_model.py    # ResNet18 + STFT architecture ✅
├── train_resnet18_stft.py    # ResNet18 STFT training pipeline ✅
├── ensemble_models.py        # Ensemble system & ResNet18+LogMel ✅
├── logmel_dataset.py         # LogMel Dataset class ✅
├── train_and_validate_classical_models_from_mfcc.py # ML baselines ✅
├── check_dir.py              # Directory utilities ✅
└── validate_segmented_dataset.py # Quality assurance ✅

# Production Scripts
├── load_trained_models.py    # Model loading system ✅
├── save_models_batch.py      # Model saving utilities ✅
├── example_usage.py          # Usage demonstrations ✅
├── ensemble_projection.py    # Performance analysis ✅
└── models/                   # Trained models directory
    ├── resnet18_logmel_full.pth     # 42.7MB - ResNet18+LogMel
    ├── random_forest_mfcc.pkl       # 6.2MB - RF+MFCC  
    ├── ensemble_config.json         # Optimized configuration
    └── *_metadata.json              # Model performance metrics
```

### Data Flow Architecture
```
Raw SPRSound Dataset → Organized (train/val/test) → Clean (no Poor Quality) → 
Segmented (individual events) → Resampled & Normalized → Trimmed (2s @ 16kHz) →
Feature Extraction (MFCC + Log-Mel + STFT) → Model Training & Evaluation
```

### Path Configuration System
- All paths centrally managed through `config.py`
- Supports multiple processing stages: raw → organized → processed → clean → segmented
- Structured for train/validation/test splits with both WAV and JSON annotation files

## Target Model Architecture (Task 1.1 Winner Analysis)

### ECE UoIT (1st place Task 1.1 - 89.26%):
- **Feature Extraction**: STFT, Mel Spectrograms, Wav2vec
- **Models**: LightCNN, Pretrained ResNet18, Audio Spectrogram Transformer (AST)
- **Class Imbalance**: Weighted Loss Function
- **Target Metrics**: SE: 88.94%, SP: 89.58%, Score: 89.26%

### NIIT (2nd place Task 1.1 - 88.86%):
- **Preprocessing**: Fixed audio length → abs(STFT) → Spectrogram clipping to 40×188
- **Model**: ResNet backbone + Fully Connected layer for binary classification
- **Class Imbalance**: Weighted Random Sampler + Focal Loss

## Dataset Characteristics

- **SPRSound**: 2,683 records, 9,089 events from 292 pediatric participants (ages 0.2-16.2 years)
- **Audio Format**: WAV files at 8kHz, 16-bit
- **Annotations**: JSON files with event-level and record-level classifications
- **Task 1-1 Focus**: Binary classification (Normal vs Adventitious events)
- **Class Distribution**: ~6,887 Normal events, ~2,202 Adventitious events

## 🏆 FINAL RESULTS - OBJECTIVE COMPLETED

**Status**: ✅ **PROJECT SUCCESSFULLY COMPLETED** - Target exceeded with **89.58%** Final Score

**Final Ensemble Results**: 
- **🎯 Ensemble SPRSound**: **89.58%** (SE: 87.91%, SP: 91.28%, Accuracy: 90.27%)
- **Target ECE UoIT**: 89.26% - **EXCEEDED by +0.32%**
- **Configuration**: RF+MFCC (80%) + ResNet18+STFT (10%) + ResNet18+LogMel (10%)

**Individual Model Results**:
- **Random Forest + MFCC**: 84.36% final score
- **ResNet18 + LogMel**: 78.56% final score (SE: 97.94%, SP: 62.98%)
- **ResNet18 + STFT**: 87.68% final score (SE: 90.75%, SP: 84.71%)

### Current Experimental Results:

#### MFCC Features (Classical ML):
- **Random Forest**: 84.36% accuracy, 86.30% precision, 59.81% recall 
- **SVM**: 82.87% accuracy, 84.62% precision, 55.70% recall
- **Logistic Regression**: 78.39% accuracy, 68.97% precision, 56.96% recall

#### Log-Mel Features (Deep Learning):
- **CNN Log-Mel**: 82.09% test accuracy, 61.10% precision, 94.09% recall

#### STFT Features (Deep Learning - ECE UoIT approach):
- **ResNet18 + STFT + Focal Loss**: 87.68% final score (Task 1.1 metrics)
  - **Training Results**: 25 epochs, final train accuracy: 97.55%, best val accuracy: 88.25%
  - **Test Performance** (optimized threshold 0.55): 
    - Sensitivity (SE): 90.75% - Strong Adventitious detection
    - Specificity (SP): 84.71% - Good Normal detection  
    - Average Score (AS): 87.73%
    - Harmonic Score (HS): 87.62%
    - **Final Score**: 87.68%
  - **Additional Metrics**: ROC-AUC: 0.9471, Average Precision: 0.8664
  - Class Weights: [0.6451, 2.2231] for balanced training
  - **Improvement over previous run**: +0.31% (from 87.37% to 87.68%)

## Evaluation Metrics

Task 1-1 uses combined scoring:
- **Sensitivity (SE)**: Adventitious detection rate
- **Specificity (SP)**: Normal detection rate  
- **Average Score (AS)**: (SE + SP) / 2
- **Harmonic Score (HS)**: 2 * SE * SP / (SE + SP)
- **Final Score**: (AS + HS) / 2

Target to exceed: **0.8926** (89.26%) - ✅ **ACHIEVED: 0.8958 (89.58%)**

## 🚀 **PRODUCTION-READY SYSTEM**

### Saved Models (Ready for Use)
All trained models are saved in `models/` directory:
- `resnet18_stft_*` - ResNet18+STFT (87.68% individual)
- `resnet18_logmel_*` - ResNet18+LogMel (78.56% individual)  
- `random_forest_mfcc.*` - Random Forest+MFCC (84.36% individual)
- `ensemble_config.json` - Optimized ensemble configuration

### Usage Instructions
```python
# Load the winning ensemble system
from load_trained_models import SPRSoundEnsemble

# Initialize and load all models
ensemble = SPRSoundEnsemble(models_dir="models")
ensemble.load_all_models()

# Make predictions on new data
results = ensemble.predict_ensemble(
    stft_loader=your_stft_loader,
    logmel_loader=your_logmel_loader,
    mfcc_features=your_mfcc_features,
    threshold=0.55  # Optimized threshold
)

predictions = results['predictions']  # Binary predictions (0=Normal, 1=Adventitious)
probabilities = results['probabilities']  # [P(Normal), P(Adventitious)]

# Expected performance: 89.58% final score on SPRSound Task 1-1
```

### Available Scripts
- `load_trained_models.py` - Complete model loading and prediction system
- `example_usage.py` - Usage examples and demonstrations  
- `ensemble_projection.py` - Performance analysis and projections
- `save_models_batch.py` - Model training and saving utilities

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/palopos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/palopos)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
