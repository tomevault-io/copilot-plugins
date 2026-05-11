## mlx-tensor-formats

> **❌ WRONG ASSUMPTION**: MLX Swift uses PyTorch-style NCHW format

# MLX Tensor Format Conventions & Common Pitfalls

## Critical: MLX Swift and Python Use Identical Tensor Formats

**❌ WRONG ASSUMPTION**: MLX Swift uses PyTorch-style NCHW format  
**✅ CORRECT**: MLX Swift uses the same NHWC format as Python MLX

### Tensor Format Rules

1. **Conv2d Input Format**: `[N, H, W, C]` (batch, height, width, channels)
2. **Conv1d Input Format**: `[N, L, C]` (batch, length, channels)  
3. **Weight Formats**: Identical between Python and Swift MLX
   - Conv2d weights: `[out_channels, kernel_h, kernel_w, in_channels_per_group]`
   - Conv1d weights: `[out_channels, kernel_size, in_channels_per_group]`

## Key Project Files

### Core Implementation
- [Sources/ParakeetMLX/Conformer.swift](mdc:Sources/ParakeetMLX/Conformer.swift) - Main conformer encoder with DwStridingSubsampling
- [Sources/ParakeetMLX/ParakeetMLX.swift](mdc:Sources/ParakeetMLX/ParakeetMLX.swift) - Main model class and weight loading
- [Sources/ParakeetMLX/RNNT.swift](mdc:Sources/ParakeetMLX/RNNT.swift) - RNN-T components (LSTM, prediction, joint networks)

### Python Reference
- [Python/conformer.py](mdc:Python/conformer.py) - Reference Python implementation
- [Python/rnnt.py](mdc:Python/rnnt.py) - Reference RNN-T implementation
- [Python/parakeet.py](mdc:Python/parakeet.py) - Main Python model interface

## Common Pitfalls & Solutions

### 1. Weight Loading (ParakeetMLX.swift)
```swift
// ❌ DON'T transpose weights - they're already in correct format
// ✅ DO use weights as-is from safetensors
let transformedWeights = weights  // No transposition needed!
```

### 2. Conv2d Tensor Handling (Conformer.swift)
```swift
// ❌ Wrong: Assuming NCHW format
// ✅ Correct: Handle NHWC format with proper transposes
x = x.transposed(axes: [0, 2, 3, 1])  // [NCHW] -> [NHWC] for Conv2d
// ... conv operations ...
x = x.transposed(axes: [0, 3, 1, 2])  // [NHWC] -> [NCHW] back
```

### 3. Depthwise Convolution Configuration
```swift
// ❌ Wrong: groups=1 for depthwise conv
// ✅ Correct: groups=inputChannels for depthwise conv
Conv2d(
    inputChannels: inChannels,
    outputChannels: inChannels,
    groups: inChannels  // Essential for depthwise convolution
)
```

### 4. LSTM Tensor Handling (RNNT.swift)
```swift
// ✅ These transposes are CORRECT for batch_first LSTM handling
if batchFirst {
    x = x.transposed(axes: [1, 0, 2])  // [batch, seq, features] -> [seq, batch, features]
}
```

## Debugging Tips

1. **Check weight shapes**: Depthwise conv weights have shape `[out_channels, kernel_h, kernel_w, 1]`
2. **Verify tensor dimensions**: Print shapes before/after each operation
3. **Compare with Python**: Use same transpose operations as Python reference
4. **Weight loading**: No format conversion needed between Python and Swift MLX

## Architecture Overview

- **Encoder**: Conformer with DwStridingSubsampling for downsampling
- **Decoder**: LSTM-based prediction network  
- **Joint**: Combines encoder and decoder outputs for final predictions
- **TDT Decoding**: Time-dependent transducer with duration prediction

---
> Source: [FluidInference/swift-parakeet-mlx](https://github.com/FluidInference/swift-parakeet-mlx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
