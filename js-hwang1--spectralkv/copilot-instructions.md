## spectralkv

> **Goal:** Implement "SpectralKV," a novel KV-cache eviction policy for LLM inference that uses asynchronous Random Walks (PageRank approximation) on the Attention Graph.

# SpectralKV (formerly ShadowKV) - Research Agent Guidelines

## 1. Project Context & Objectives
**Goal:** Implement "SpectralKV," a novel KV-cache eviction policy for LLM inference that uses asynchronous Random Walks (PageRank approximation) on the Attention Graph.
**Target:** ICML 2026 Systems/ML Track. Code must be publication-ready, reproducible, and highly optimized.
**Stack:**
- **Core Logic:** C++17, CUDA 12.x (Kernels).
- **Interface:** PyTorch (via C++ Extensions / `torch.utils.cpp_extension`), Hugging Face Transformers.
- **Analysis:** Python (NetworkX, Matplotlib, NumPy).

## 2. Critical Architecture Rules
1.  **Asynchronous by Design:** The `ShadowWalker` kernel must *never* block the main generation stream. Always use `cudaStream_t` explicitly.
2.  **Memory Layout:**
    - Use `struct-of-arrays` (SoA) or coalesced `array-of-structs` (AoS) for the `GraphCache`.
    - All pointer arguments in CUDA kernels must be marked `__restrict__`.
3.  **Decoupling:**
    - `Monitor` logic (PageRank/Random Walk) must be decoupled from `Eviction` logic (Mask application).
    - The Python prototype (Phase 1) should be a standalone script, not mixed with CUDA code.

## 3. C++ / CUDA Coding Standards
- **Style:** Google C++ Style Guide.
- **Performance:**
    - Avoid `shared_ptr` inside kernels. Use raw pointers with strict lifecycle management in host code.
    - Use `__forceinline__` for helper device functions.
    - Prefer `fast_rand` (XORWOW/PCG) over heavy curand states if possible.
- **Safety:**
    - Always check `cudaGetLastError()` after kernel launches.
    - Use `AT_DISPATCH_FLOATING_TYPES_AND_HALF` for PyTorch bindings to support FP16/BF16.
- **Naming:**
    - Kernels: `snake_case_kernel` (e.g., `shadow_walker_kernel`).
    - Host Classes: `PascalCase` (e.g., `EvictionMonitor`).
    - Variables: `snake_case`.

## 4. Python Coding Standards
- **Style:** Black formatter, strict Type Hints (`def func(x: torch.Tensor) -> float:`).
- **Research Hygiene:**
    - **Configuration:** Use `dataclasses` or `hydra` for configs. NO hardcoded magic numbers (e.g., `budget=4096`).
    - **Reproducibility:** Every script must have a `set_seed(seed)` function that locks torch, cuda, and numpy seeds.
    - **Plotting:** All plots must use `seaborn` or `matplotlib` with "publication" settings (PDF output, high DPI, clear legends).
- **Dependencies:**
    - Do not modify `transformers` library code directly. Subclass `LlamaForCausalLM` or use Forward Hooks.

## 5. Build & Environment
- **Build System:** `ninja` is preferred.
- **Extension:** Use `setuptools` with `CppExtension` and `CUDAExtension`.
- **Commands:**
    - Build: `pip install -e .`
    - Test: `pytest tests/ --verbose`
    - Profile: `nsys profile python run_benchmark.py`

## 6. Definition of "Done"
A task is not complete until:
1.  It compiles without warnings (`-Wall -Wextra`).
2.  It includes a specific unit test covering edge cases (e.g., empty cache, full cache).
3.  (For Kernels) It includes a comment block explaining the block/thread mapping and shared memory usage.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Js-Hwang1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
