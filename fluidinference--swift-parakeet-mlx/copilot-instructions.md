## parakeet-mlx-swift-overview

> This project is a Swift implementation of the Parakeet speech recognition models using MLX Swift, converted from the original Python MLX implementation.

# ParakeetMLX Swift - Project Overview

This project is a Swift implementation of the Parakeet speech recognition models using MLX Swift, converted from the original Python MLX implementation.

## Core Architecture

The main library is structured around these key components:

- **[ParakeetMLX.swift](mdc:Sources/ParakeetMLX/ParakeetMLX.swift)** - Main library interface with `ParakeetTDT` model class, streaming support, and model loading functions
- **[Conformer.swift](mdc:Sources/ParakeetMLX/Conformer.swift)** - Conformer encoder implementation with attention mechanisms, convolution modules, and positional encoding
- **[RNNT.swift](mdc:Sources/ParakeetMLX/RNNT.swift)** - RNN-T decoder components including prediction network, joint network, and LSTM/GRU implementations
- **[AudioProcessing.swift](mdc:Sources/ParakeetMLX/AudioProcessing.swift)** - Audio preprocessing pipeline with STFT, mel-scale conversion, and windowing functions

## Main Interfaces

### Model Loading
```swift
let model = try loadParakeetModel(from: "nvidia/parakeet-tdt_ctc-1.1b", dtype: .bfloat16)
```

### Basic Transcription
```swift
let result = try model.transcribe(audioData: audioData)
print("Transcription: \(result.text)")
```

### Streaming Transcription
```swift
let stream = model.transcribeStream(contextSize: (256, 256), depth: 1)
try stream.addAudio(audioChunk)
let partialResult = stream.result
```

## Package Structure

- **[Package.swift](mdc:Package.swift)** - Swift package definition with MLX Swift dependencies
- **[Sources/ParakeetCLI/main.swift](mdc:Sources/ParakeetCLI/main.swift)** - Command-line interface for transcription
- **[Tests/ParakeetMLXTests/](mdc:Tests/ParakeetMLXTests/)** - Unit tests for core functionality
- **[Examples/BasicTranscription.swift](mdc:Examples/BasicTranscription.swift)** - Usage examples and integration patterns
- **[README-Swift.md](mdc:README-Swift.md)** - Comprehensive documentation and API reference

## Key Features

- High-performance speech recognition optimized for Apple Silicon
- Real-time streaming inference with configurable context windows
- Chunked processing for long audio files
- Word-level timestamp alignment
- Support for Hugging Face Hub model loading
- Cross-platform compatibility (macOS/iOS)

---
> Source: [FluidInference/swift-parakeet-mlx](https://github.com/FluidInference/swift-parakeet-mlx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
