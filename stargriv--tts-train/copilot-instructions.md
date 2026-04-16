## tts-train

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Russian Text-to-Speech (TTS) system trained on 228 Russian audio files containing phrases from an Agatha Christie story about Hercule Poirot. The text content is encoded in the audio filenames using underscores as word separators.

## Architecture

The TTS system uses a Tacotron2-inspired architecture with Russian-specific text processing:

1. **RussianTextProcessor** (model.py): Handles character-level encoding of Cyrillic text with 78-character vocabulary including full Russian alphabet and punctuation
2. **TextEncoder**: Convolutional layers + bidirectional LSTM for text embedding
3. **Attention**: Cross-attention mechanism between decoder and encoder states
4. **AudioDecoder**: LSTM-based decoder that generates mel spectrograms with pre-net regularization and stop token prediction
5. **TTSDataset** (train.py): Custom dataset class that loads audio files and converts them to mel spectrograms using librosa

## Essential Commands

### Environment Setup
```bash
python3 -m venv venv
source venv/bin/activate  # macOS/Linux
pip install -r requirements.txt
```

### Dataset Preparation
```bash
python prepare_dataset.py
```
Creates training metadata from audio filenames in `input/` directory, generating `data/train.csv`, `data/val.csv`, and statistics.

### Training
```bash
python train.py --epochs 100 --batch_size 8 --device auto
```
Device selection automatically prioritizes Apple Silicon MPS, then CUDA, then CPU. Training saves checkpoints to `checkpoints/` directory.

### Testing Setup
```bash
python test_setup.py
```
Comprehensive test of device support, text processing, model creation, dataset loading, and inference setup.

### Inference
```bash
# Single text
python inference.py --text "Привет мир" --output hello.wav

# Interactive mode
python inference.py

# Batch processing
python inference.py --text_file texts.txt --output_dir generated/
```

## Key Implementation Details

### Device Support
The codebase is optimized for Apple Silicon with automatic MPS (Metal Performance Shaders) detection. Training scripts check for MPS availability before falling back to CUDA or CPU.

### Audio Processing
- Sample rate: 22,050 Hz
- Mel channels: 80
- Uses librosa for mel spectrogram conversion
- Griffin-Lim algorithm for mel-to-audio conversion during inference
- Audio files are normalized and converted to log-scale mel spectrograms

### Training Pipeline
- Teacher forcing during training with ground truth mel spectrograms
- Attention weights visualization available
- Stop token prediction for variable-length output
- Best model saved based on validation loss
- Training curves plotted and saved

### Text Processing
Russian text is converted to lowercase and mapped to character IDs. Unknown characters are replaced with spaces. The vocabulary includes all Cyrillic letters (both cases), punctuation, and spaces.

### Dataset Structure
Audio files in `input/` directory use filename format: `{number}_{russian_text_with_underscores}.wav`
Processed metadata is stored in `data/` directory with train/validation splits (90%/10%).

## Development Notes

The model architecture expects encoder output dimension (256) to match the encoder_dim parameter in the Attention module. The decoder dimension (1024) is separate and configurable.

When adding new audio files, ensure filenames follow the existing pattern with Russian text encoded using underscores as word separators. Re-run `prepare_dataset.py` to update metadata files.

The inference system can work with untrained models (will show warnings) but requires proper checkpoint files for quality speech synthesis.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stargriv) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
