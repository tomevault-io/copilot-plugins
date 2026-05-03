## gpu-first-pipeline

> GPU-first data pipeline — WebGPU is required, no needless CPU copies anywhere in the stack


# GPU-First Data Pipeline

SpawnScene is a high-performance Gaussian Splatting application. Performance and visual quality are the primary goals. **WebGPU is a hard minimum requirement.** **SpawnDev.ILGPU is the primary tool for all GPU compute.** Every stage of the pipeline must be GPU-resident.

## WebGPU Is Required — No Fallbacks

Do NOT write fallback paths for WebGL, Wasm, or CPU. If WebGPU is unavailable, the app throws a clear error. This means:
- `GpuService` initializes WebGPU only — no `AllAcceleratorsAsync()`, no backend selection logic
- `_gpu.Accelerator` is always a `WebGPUAccelerator` — cast directly, do not use `is`/`as` checks
- `ExternalWebGPUMemoryBuffer` is always available — use it freely
- `ICanvasRenderer` is always `WebGPUCanvasRenderer` — instantiate directly if needed, no factory fallback logic required

```csharp
// ✅ CORRECT: WebGPU-only init
await builder.WebGPU();
_context = builder.ToContext();
_accelerator = (WebGPUAccelerator)await _context.GetDevices<WebGPUILGPUDevice>()[0]
    .CreateAcceleratorAsync(_context, null);

// ❌ WRONG: fallback logic that will never be needed
_accelerator = await _context.CreatePreferredAcceleratorAsync(); // don't use
```

## Core Rule: No Needless CPU Copies

Data must never leave the GPU unless there is no alternative. Before any readback ask: _can an ILGPU kernel or WebGPU shader do this instead?_

Acceptable CPU transfers (must have a `// CPU transfer: <reason>` comment):
- File I/O (images, PLY, SPLAT — unavoidable source of data)
- Scalar metadata only (e.g. 2 floats min/max for UI display)

Everything else stays on GPU.

## The Full GPU Pipeline

```
Image file (CPU read, unavoidable)
  → Upload RGBA once → GPU                        [CPU→GPU, file I/O]
  → ILGPU PreprocessKernel (RGBA→NCHW 518×518)   [GPU]
  → ort.Tensor.fromGpuBuffer (ORT input)          [GPU, zero-copy]
  → ONNX WebGPU inference                        [GPU]
  → outputTensor.GPUBuffer (ORT output)           [GPU, zero-copy]
  → ExternalWebGPUMemoryBuffer wrapper            [GPU, zero-copy]
  → ILGPU ResizeDepthKernel (518×518→origW×H)    [GPU]
  → MinDepth/MaxDepth → CPU (8 bytes, UI only)    [GPU→CPU, justified]
  → ILGPU UnprojectAndPackKernel                 [GPU]
  → GpuSplatSorter (radix sort, ILGPU 3.5.0)     [GPU]
  → GpuGaussianRenderer (WGSL shaders)            [GPU]
  → Canvas                                        [GPU→display]
```

## Key APIs

- `ExternalWebGPUMemoryBuffer(accel, gpuBuffer, count, elementSize)` — zero-copy wrap of any external `GPUBuffer`
- `new WebGPUCanvasRenderer(webGpuAccel)` — direct GPU→canvas present, no CPU readback
- `_ort.TensorFromGpuBuffer(gpuBuffer, options)` — create ONNX input tensor from GPU buffer
- `outputTensor.GPUBuffer` — access ONNX output directly on GPU

## Anti-Patterns

```csharp
// ❌ copies GPU tensor to CPU
var data = await outputTensor.GetDataAsync<Float32Array>();

// ✅ stays on GPU
var externalBuf = new ExternalWebGPUMemoryBuffer(accel, outputTensor.GPUBuffer, n, 4);

// ❌ CPU packing loop + upload
for (int i = 0; i < n; i++) packed[i*10] = ...; accelerator.Allocate1D(packed);

// ✅ ILGPU kernel writes packed output on GPU, feed buffer directly to sorter

// ❌ CPU colorization → ImageData
byte[] rgba = ColorizeDepthMap(dr); ctx.PutImageData(new ImageData(rgba), 0, 0);

// ✅ ILGPU colorization kernel → new WebGPUCanvasRenderer(accel).PresentAsync(buf)
```

## DepthResult Must Not Contain CPU Arrays

`DepthResult` holds only GPU-resident `MemoryBuffer1D<float>` for raw depth and scalar metadata. No `float[]` arrays. It is `IDisposable`.

---
> Source: [LostBeard/SpawnScene](https://github.com/LostBeard/SpawnScene) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
