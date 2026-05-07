## gpu

> GPU backend guidelines for skia-rs-gpu


# GPU Backend Guidelines

## Feature Flags

```toml
[features]
default = ["wgpu-backend"]
vulkan = ["dep:ash"]
opengl = ["dep:glow"]
wgpu-backend = ["dep:wgpu"]
```

## Backend Abstraction

```rust
pub trait GpuBackend: Send + Sync {
    fn create_surface(&self, info: &ImageInfo) -> Result<GpuSurface, GpuError>;
    fn flush(&self);
}
```

## Supported Backends

- **wgpu**: Cross-platform WebGPU abstraction (default)
- **Vulkan**: Low-level via `ash` crate
- **OpenGL**: Via `glow` crate
- **Metal**: macOS/iOS (planned)

## Resource Management

- Use GPU resource pools for frequently allocated objects
- Implement proper cleanup in `Drop` traits
- Handle device lost scenarios gracefully

---
> Source: [quinnjr/skia-rs](https://github.com/quinnjr/skia-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
