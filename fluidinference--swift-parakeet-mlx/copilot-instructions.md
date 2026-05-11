## mlx-swift-api-conventions

> This rule documents important MLX Swift API patterns and common conversion issues from Python MLX to Swift MLX.

# MLX Swift API Conventions and Common Pitfalls

This rule documents important MLX Swift API patterns and common conversion issues from Python MLX to Swift MLX.

## Core API Patterns

### Array Operations
- Use `MLX.concatenated()` instead of `MLXArray.concatenated()`
- Use `MLX.stacked()` instead of `MLXArray.stacked()`
- Use `MLX.padded()` instead of `MLX.pad()`

### Padding Operations
```swift
// CORRECT: Use MLX.padded with proper parameter labels and types
x = MLX.padded(x, widths: padArray.map { IntOrPair($0) }, mode: .constant, value: MLXArray(0.0))

// WRONG: Missing widths label, wrong value type
x = MLX.padded(x, padArray, mode: .constant, value: 0.0)
```

### Type Conversion
```swift
// CORRECT: Specify type explicitly for .item()
let length = Int(array.item(Int32.self))
let value = array.item(Float.self)

// WRONG: Ambiguous type conversion
let length = Int(array.item())  // Compilation error
```

### IntOrPair Usage
```swift
// CORRECT: Pass tuple as single parameter
kernelSize: IntOrPair((3, 3))
stride: IntOrPair((2, 2))

// WRONG: Pass two separate parameters
kernelSize: IntOrPair(3, 3)  // Compilation error
```

## Neural Network Layers

### Conv2d Parameters
```swift
///
/// ### See Also
/// - <doc:convolution>
/// - ``IntOrPair``
/// - ``conv1d(_:_:stride:padding:dilation:groups:stream:)``
/// - ``conv3d(_:_:stride:padding:dilation:groups:stream:)``
/// - ``convolve(_:_:mode:stream:)``
/// - ``convGeneral(_:_:strides:padding:kernelDilation:inputDilation:groups:flip:stream:)-9t1sj``
public func conv2d(
    _ array: MLXArray, _ weight: MLXArray, stride: IntOrPair = 1, padding: IntOrPair = 0,
    dilation: IntOrPair = 1, groups: Int = 1, stream: StreamOrDevice = .default
) -> MLXArray {
    var result = mlx_array_new()
    mlx_conv2d(
        &result,
        array.ctx, weight.ctx, stride.first.int32, stride.second.int32, padding.first.int32,
        padding.second.int32, dilation.first.int32, dilation.second.int32, groups.int32,
        stream.ctx)
    return MLXArray(result)
}
```

### Linear Layers
```swift
// Standard linear layer
let linear = Linear(inputDim, outputDim, bias: true)
```

### Activation Functions
Use MLX namespace for activation functions:
```swift
// CORRECT
let output = MLX.sigmoid(x)
let output = MLX.tanh(x)
let output = MLX.softmax(x, axis: -1)

// Custom activations
private func silu(_ x: MLXArray) -> MLXArray {
    return x * MLX.sigmoid(x)
}
```

## FFT Operations
```swift
// CORRECT: Use MLX.rfft
let fftResult = MLX.rfft(frameMatrix, axis: -1)

// WRONG: MLX.FFT.rfft doesn't exist
let fftResult = MLX.FFT.rfft(frameMatrix, axis: -1)
```

## Common Conversion Patterns

### From Python mx.pad to Swift MLX.padded
```python
# Python MLX
x = mx.pad(x, pad_width, mode='constant', constant_values=0.0)
```

```swift
// Swift MLX
x = MLX.padded(x, widths: padWidth.map { IntOrPair($0) }, mode: .constant, value: MLXArray(0.0))
```

### From Python mx.concatenate to Swift MLX.concatenated
```python
# Python MLX
result = mx.concatenate([array1, array2], axis=0)
```

```swift
// Swift MLX
result = MLX.concatenated([array1, array2], axis: 0)
```

## Best Practices

1. **Always specify types for .item()** - MLX Swift requires explicit type specification
2. **Use IntOrPair correctly** - Pass tuples as single parameters, not separate arguments
3. **Wrap scalar values in MLXArray** - Many functions expect MLXArray parameters, not raw scalars
4. **Use MLX namespace** - Most functions are in the MLX namespace, not as instance methods

---
> Source: [FluidInference/swift-parakeet-mlx](https://github.com/FluidInference/swift-parakeet-mlx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
