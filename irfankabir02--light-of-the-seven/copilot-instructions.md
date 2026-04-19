## light-of-the-seven

> Motion-centric constraint rules for Python-Rust integration pipeline.

# GRID Platform Integration Rules

Motion-centric constraint rules for Python-Rust integration pipeline.

## Overview

This document defines motion-centric constraints that translate momentum and magnitude values between Python constraints (`.cursor/rules/`) and Rust type contracts. These rules complement the `.cursor` rules and focus on dynamic system behavior, quantization, and locomotion.

## Motion Constraints

### Momentum Thresholds

- **Minimum Velocity**: `0.0` - System must have forward progress
- **Maximum Velocity**: `1.0` - Prevent runaway processes
- **Minimum Acceleration**: `-1.0` - Allow controlled deceleration
- **Maximum Acceleration**: `1.0` - Prevent abrupt changes

### Magnitude Ranges

- **Minimum Amplitude**: `0.0` - No negative magnitudes
- **Maximum Amplitude**: `10.0` - Bounded system scale

## Quantization Level Mappings

Quantization levels define precision for metric translation between Python and Rust:

| Level | Step Size | Precision | Use Case |
|-------|-----------|-----------|----------|
| `coarse` | 1.0 | 1 decimal | Quick approximations, initial estimates |
| `medium` | 0.1 | 2 decimals | Balanced precision (default) |
| `fine` | 0.01 | 4 decimals | Detailed analysis |
| `ultra_fine` | 0.001 | 6 decimals | High-precision measurements |

### Quantization Rules

1. All cognitive metrics must be quantized before translation to Rust
2. Use `medium` quantization by default
3. Use `fine` for decision quality metrics
4. Use `coarse` for initial load estimates

## Motion Direction Constraints

### Forward Motion
- Velocity > 0.0: System progressing forward
- Acceleration >= 0.0: Accelerating or maintaining speed

### Backward Motion
- Velocity < 0.0: System regressing (error state)
- Acceleration < 0.0: Controlled deceleration

### Stabilization
- Velocity ≈ 0.0: System stable
- Acceleration ≈ 0.0: No change in momentum

## Complementary Relationship Rules

### Python Constraints → Rust Types

1. **Type Mapping**:
   - Python `float` → Rust `f64`
   - Python `int` → Rust `i32`
   - Python `str` → Rust `String`
   - Python `List[T]` → Rust `Vec<T>`
   - Python `Dict[K, V]` → Rust `HashMap<K, V>`

2. **Range Constraints**:
   - Python validation rules map to Rust type bounds
   - Quantized values must fit Rust numeric types

3. **Enum Mapping**:
   - Python string enums → Rust enum variants
   - Enum values must match exactly (case-sensitive)

### Constraint Translation Process

1. Load Python constraints from `.cursor/rules/`
2. Apply quantization using specified level
3. Validate against momentum/magnitude constraints
4. Translate to Rust type contracts
5. Validate Rust compilation (definition-based handshake)

## Cognitive Metrics Constraints

### Load Metrics

- **Estimated Load**: Range [0.0, 10.0], quantized to `medium`
- **Complexity Score**: Range [0.0, 1.0], quantized to `fine`
- **Function Density**: Range [0.0, 1.0], quantized to `medium`

### Decision Metrics

- **Decision Quality**: Range [0.0, 1.0], quantized to `fine`
- **Integration Score**: Range [0.0, 1.0], quantized to `medium`
- **System Coverage**: Range [0.0, 1.0], quantized to `coarse`

### Alignment Metrics

- **Alignment Score**: Range [0.0, 1.0], quantized to `fine`
- **Coupling Score**: Range [0.0, 1.0], quantized to `medium`

## Validation Rules

1. All metrics must pass momentum validation before export
2. Quantization level must be valid (coarse, medium, fine, ultra_fine)
3. Type translations must be verified by Rust compiler
4. Enum values must match between Python and Rust exactly

## Integration Checkpoints

The integration pipeline validates at these checkpoints:

1. **Schema Validation**: Artifact structure matches expected format
2. **Type Validation**: Python types match Rust type contracts
3. **Constraint Validation**: Metrics pass momentum/magnitude constraints
4. **Compilation Check**: Rust code compiles successfully (`cargo build`)
5. **Runtime Check**: Rust binary executes (`cargo run`)

Each checkpoint must pass before proceeding to the next stage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/irfankabir02) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
