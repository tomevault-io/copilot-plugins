## nunchaku-zit-lora

> This project implements the Node-Slot-Link design pattern for ML inference pipelines.

# Agent Instructions

## Project Architecture

This project implements the Node-Slot-Link design pattern for ML inference pipelines.

## Design References

When working on this codebase, follow patterns from:

### ComfyUI
https://github.com/comfyanonymous/ComfyUI
- **Ordered slots**: All inputs/outputs accessed by index
- **String types**: Type system uses string comparison only
- **Lazy loading**: Import and load data only when needed
- **Node structure**: Execution units with defined input/output slots

### LiteGraphJS
https://github.com/jagenjo/litegraph.js
- **Graph execution**: Node-based execution model
- **Slot connections**: Links connect compatible typed slots
- **Visual programming**: Support for node editor paradigm

### HuggingFace Diffusers
https://github.com/huggingface/diffusers
- **Use their primitives**: Extend diffusers components, don't replace
- **Follow their patterns**: Pipeline structure, model loading
- **LoRA support**: Use built-in LoRA weight management

## Core Constraints

1. **Single Base Model**: Project supports one base model (Z-Image-Turbo)
2. **No Custom Features**: Don't add features that extend packages without permission
3. **Framework First**: Always use framework primitives before custom code
4. **Ordered Returns**: Functions return tuples `(val1, val2, ...)` in slot order
5. **String Types Only**: No runtime type checking beyond string comparison

## Key Classes

- `SlotData`: Base class for ordered data structures
- `Node`: Base class for execution units with slots
- `LoRASlot`: Ordered LoRA configuration data

## Specifications

See `docs/` directory for:
- Architecture specifications (RFC-ARCH-001)
- Project specifications (PROJECT-SPEC-001)
- Design patterns (PATTERN-Node-Slot-Link)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/electblake)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/electblake)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
