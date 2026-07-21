## nndeploy

> This file provides guidance to codex when working with code in this repository.

# AGENTS.md

This file provides guidance to codex when working with code in this repository.

## Project Overview

nndeploy is a high-performance AI deployment framework for edge computing (desktop, mobile, edge devices, servers). It provides a visual workflow editor and supports 13+ inference backends including ONNXRuntime, TensorRT, OpenVINO, MNN, TNN, ncnn, CoreML, AscendCL, RKNN, SNPE, TVM, PyTorch, and an internal inference engine.

## Build Commands

### Basic Build (Linux/macOS)

```bash
mkdir build && cd build
cp ../cmake/config.cmake .
# Edit config.cmake to enable desired backends
cmake ..
make -j
make install
```

The build creates:
- `libnndeploy_framework.so` (or `.dll`/`.dylib`) - Main framework library
- `libnndeploy_plugin_*.so` - Algorithm plugins
- `nndeploy_demo_*` - Demo executables
- `build/nndeploy_${VERSION}_${PLATFORM}_...` - Install directory

### Build Configuration

Edit `build/config.cmake` to enable/disable components. Key options:
- `ENABLE_NNDEPLOY_BUILD_SHARED` - Build shared library (default ON)
- `ENABLE_NNDEPLOY_DEVICE_CUDA` - Enable NVIDIA CUDA
- `ENABLE_NNDEPLOY_DEVICE_ASCEND_CL` - Enable Huawei Ascend
- `ENABLE_NNDEPLOY_INFERENCE_*` - Enable specific inference backends (all OFF by default)
- `ENABLE_NNDEPLOY_PLUGIN_*` - Enable algorithm plugins

### Python Build

After C++ build:
```bash
cd python
pip install -e .
```

Python bindings require C++17 ABI for tokenizer-cpp support.

### Test Execution

Tests use GoogleTest framework and are located in `test/`:
```bash
cd build
ctest --verbose
```

Run individual tests: `./test/edge_test`, `./test/graph_test`

## Architecture

### Core Modules

Located in `framework/include/nndeploy/` and `framework/source/nndeploy/`:

1. **base/** - Utilities: Status, logging, memory tracking, time profiler, file operations
2. **thread_pool/** - Thread pool for parallel task execution
3. **device/** - Device abstraction (CPU, CUDA, OpenCL, Metal, AscendCL), memory management, Tensor, Buffer, MemoryPool
4. **op/** - High-performance operators (CPU/x86/ARM/CUDA/Ascend C kernels)
5. **ir/** - Intermediate Representation (heavily ONNX-inspired), ModelDesc with OpDesc/ValueDesc, serialization to JSON/safetensors
6. **net/** - Neural network graph representation
7. **inference/** - Multi-backend inference abstraction (default, tensorrt, openvino, mnn, tnn, ncnn, coreml, onnxruntime, etc.)
8. **dag/** - Directed Acyclic Graph: Node, Edge, Graph, Executor (sequential, parallel pipeline, parallel task, condition), workflow execution

### Plugin System

Located in `plugin/source/nndeploy/`:

**Templates:**
- `template/` - Base classes for creating custom nodes (C++ and Python)

**Algorithm Plugins:**
- `preprocess/` - Image preprocessing nodes
- `infer/` - Inference template node (handles single/multi I/O, static/dynamic shapes)
- `codec/` - Image/video encode/decode
- `tokenizer/` - Text tokenization (Python/C++ implementations)
- `classification/` - Classification algorithms
- `detect/` - Object detection (YOLO, DETR)
- `segment/` - Image segmentation (Segment Anything, RMBG)
- `matting/` - Image matting
- `track/` - Object tracking (FairMOT)
- `llm/`, `qwen/` - Large language models
- `stable_diffusion/` - Text-to-image generation
- `super_resolution/` - Super resolution
- `ocr/` - OCR (Paddle OCR)

### Data Flow

Workflow: **Preprocess → Infer → Postprocess**
- DAG (Directed Acyclic Graph) orchestrates the pipeline
- Edges pass data (Tensor, Mat, custom types) between Nodes
- Executors control execution mode: serial, pipeline parallel, task parallel

## Key Patterns

### Adding a New Inference Backend

1. Add enum types in `base/common.h`: `ModelType`, `InferenceType`
2. Add error codes in `base/status.h`: `StatusCode`
3. Create param class inheriting `InferenceParam` in `include/nndeploy/inference/xxx/`
4. Create inference class inheriting `Inference` in `include/nndeploy/inference/xxx/`
5. Create converter for Tensor/data conversion in `include/nndeploy/inference/xxx/`
6. Update `CMakeLists.txt` with source files and `ENABLE_NNDEPLOY_INFERENCE_XXX` option
7. Create `cmake/xxx.cmake` for library linking

### Adding a Custom Node

**C++ Node:**
- Inherit from `dag::Node` (or use template in `template/cpp/`)
- Implement `init()`, `deinit()`, `process()`, `run()`
- Register in plugin config: see `plugin/source/nndeploy/*/config.cmake`

**Python Node:**
- Use `template/python/template.py` as reference
- Create class with `run()` method
- Load via JSON workflow or Python API

### Workflow JSON Format

Saved workflows contain complete pipeline definition:
- Nodes with types and parameters
- Edges connecting nodes
- Can be loaded via `graph.load_file("workflow.json")`

### Device Memory Model

- `device::Device` - Abstract device interface
- `device::Buffer` - Unified memory container across devices
- `device::Tensor` - Tensor data structure for inference
- `device::MemoryPool` - Memory pool for efficient allocation

Zero-copy optimization is prioritized when possible.

## Dependencies

Third-party libraries in `third_party/`:
- **gflags** - Command line flags (test/demo only)
- **googletest** - Testing framework
- **pybind11** - Python bindings
- **rapidjson** - JSON parsing
- **onnx** - ONNX format support (optional, for IR)
- **protobuf** - Protocol buffers (ONNX dependency)
- **safetensors-cpp** - Safe tensor serialization
- **tokenizers-cpp** - C++ tokenizer (requires Rust, precompile required before enabling)

## Common Development Tasks

### Running a demo

Demos are in `demo/*/demo.cc`. After build:
```bash
cd build/nndeploy_*/demo
./nndeploy_demo_detect --model_path=... --image_path=...
```

### Running a JSON workflow

```bash
nndeploy-demo-run-json --json_file path/to/workflow.json
# Or Python:
python -c "import nndeploy; graph = nndeploy.dag.Graph(''); graph.load_file('workflow.json')"
```

### Debugging

- Enable `ENABLE_NNDEPLOY_TIME_PROFILER` for timing
- Enable `ENABLE_NNDEPLOY_GPU_MEM_TRACKER` for GPU memory tracking
- `ENABLE_NNDEPLOY_ADDRESS_SANTIZER` for memory error detection

## Code Organization Notes

- Headers in `framework/include/nndeploy/`
- Sources in `framework/source/nndeploy/`
- Plugin headers in `plugin/include/nndeploy/`
- Plugin sources in `plugin/source/nndeploy/`
- Python bindings in `python/src/nndeploy/`
- Demos in `demo/`
- Tests in `test/`

## Platform-Specific Notes

- **macOS**: `CMAKE_OSX_DEPLOYMENT_TARGET = 10.15`
- **Windows**: Use MSVC, `/utf-8` for UTF-8 source files
- **Android**: Cross-compile with NDK toolchain
- **Linux**: `-march=x86-64` for x86_64, `-march=armv8-a` for ARM64

---
> Source: [nndeploy/nndeploy](https://github.com/nndeploy/nndeploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
