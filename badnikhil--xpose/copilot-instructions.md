## xpose

> C++20 Vulkan compute runtime with CUDA-like ergonomics. Kernels come from

# Xpose — agent notes

C++20 Vulkan compute runtime with CUDA-like ergonomics. Kernels come from
clspv-compiled SPIR-V. Full design docs + decision log live in
`agent-docs/` (read `00-INDEX.md` first; `xpose-core.md` and
`xpose-program-launch.md` cover this repo).

## Rules
- Submodule remotes point at their public upstreams — never push those.
- Don't move submodule pins (`third_party/*` SHAs) without recording why here
  and in `agent-docs/environment.md`.
- `agent-docs/` is the shared knowledge base — committed in-repo; keep it
  updated with every change/decision/finding.
- **Found a failing edge case (especially on real hardware)? Fix the code, keep/add
  a regression test, AND document the lesson here** — don't just patch and move on.
  A desktop-only green run is NOT proof of device correctness (see coherency note).

## Third-party layout (git submodules at `third_party/`)
- After a fresh clone: `git submodule update --init --recursive`.
- Submodules: `Vulkan-Headers`, `volk`, `VulkanMemoryAllocator`, `googletest`
  (build/test deps); `SPIRV-Tools` (reference clone); `clspv` (kernel
  compiler); `clvk`, `kompute` (reference reading only, not built).
- **clspv's own deps are NOT submodules of clspv** — they're fetched clones:
  run `python3 utils/fetch_sources.py --shallow` inside `third_party/clspv`
  to populate `third_party/clspv/third_party/{llvm,SPIRV-Headers,SPIRV-Tools}`
  (~2.9 GB). `.gitmodules` sets `ignore = untracked` for clspv accordingly.
- `third_party/spirv-tools-build/` is a **gitignored out-of-tree build** of
  clspv's pinned SPIRV-Tools providing `tools/spirv-{as,dis,val}` for the
  fixture pipeline. Rebuild:
  ```sh
  cmake -G Ninja -S third_party/clspv/third_party/SPIRV-Tools \
        -B third_party/spirv-tools-build -DCMAKE_BUILD_TYPE=Release \
        -DSPIRV-Headers_SOURCE_DIR=$PWD/third_party/clspv/third_party/SPIRV-Headers \
        -DSPIRV_SKIP_TESTS=ON
  ninja -C third_party/spirv-tools-build -j6 spirv-as spirv-dis spirv-val
  ```
  (The top-level `SPIRV-Tools` submodule does NOT build against clspv's
  SPIRV-Headers pin — always use the clspv-pinned combo above.)

## Build & test
```sh
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release   # gcc default; clang also supported
cmake --build build
./build/tests/xpose_tests                                  # default device (first discrete GPU)
for d in llvmpipe RENOIR NVIDIA; do XPOSE_DEVICE=$d ./build/tests/xpose_tests; done  # full matrix
```
- **The matrix above is machine-specific.** RENOIR/NVIDIA exist only on the
  x86_64 box (`agent-docs/environment.md`). On the **aarch64 ThinkPad** there is
  one GPU + lavapipe, and the equivalent matrix is:
  ```sh
  for d in llvmpipe Adreno; do XPOSE_DEVICE=$d ./build/tests/xpose_tests; done
  ```
  That laptop's GPU is an **Adreno X1-85** (Mesa turnip) — **82/82 green on it**.
  Adreno-family coverage without a phone; see the RESOLUTION section of
  `environment.md`. NOT a substitute for the on-device gate: the laptop runs
  turnip, phones run Qualcomm's proprietary driver, and driver differences are
  exactly where the Mali bug lived.
- Third-party deps resolve to `third_party/` in-repo; override with `-DXPOSE_THIRD_PARTY_DIR=`.
- No system Vulkan SDK on this machine: headers are vendored, volk dlopens `libvulkan.so.1`.
- **On-device (Android):** `scripts/run-android.sh` cross-builds arm64, pushes the
  test binary + `tests/kernels/*.spv`, and runs the suite on a connected phone over
  adb (`--build` to recompile, `-s <serial>` to select a device). This is the ONLY
  way to exercise the non-coherent-memory path — run it before trusting buffer
  transfers. Runner details in `agent-docs/android-device-testing.md`.
  - The script's `DEFAULT_NDK` points at the x86_64 machine's path. On the
    ThinkPad override it: `ANDROID_NDK=/home/nikhil/Android/Sdk/ndk-r27c`.

### Android cross-build on the aarch64 ThinkPad
The NDK ships **x86_64-only host binaries**, so on this laptop they run under
`qemu-user` binfmt. Verified working — Xpose cross-builds in ~1m13s.
```sh
export QEMU_LD_PREFIX=/home/nikhil/x86_64-sysroot   # REQUIRED, else every NDK tool
                                                     # dies on the missing x86_64 loader
NDK=/home/nikhil/Android/Sdk/ndk-r27c
cmake -S . -B build-android -G Ninja \
  -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake \
  -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-26 -DCMAKE_BUILD_TYPE=Release
cmake --build build-android
```
`~/x86_64-sysroot` is a local unpack of the amd64 libc6/libgcc-s1/libstdc++6/zlib1g
debs — no sudo, no multiarch. `dpkg --add-architecture amd64` does NOT work here:
the arm64 archive does not serve amd64 packages. Full recipe and rationale in the
RESOLUTION section of `agent-docs/environment.md`.

## Kernels & fixtures
- Test fixtures: `tests/kernels/*.cl` + committed `.spv`. `regenerate.py`
  re-derives the binaries and now prefers the **locally built clspv** at
  `third_party/clspv/build/bin/clspv` — no network, no rate limit when several
  agents regenerate at once, and reproducible. It falls back to godbolt's hosted
  clspv if the local binary is absent; `--godbolt` forces the hosted path.
  Verified 2026-07-19: locally compiled modules load through `Program` with
  identical kernel names and workgroup sizes, and the suite passes 82/82 against
  locally regenerated fixtures, despite the local build being a different LLVM
  revision and emitting differently sized binaries.
  `tests/kernels/README.md` records the flag set, which is unchanged.
- The supported kernel ABI is fixed by those flags: storage-buffer args +
  `-pod-pushconstant` PODs + `-uniform-workgroup-size`. Program throws on
  anything else (images/samplers/UBO-PODs/implicit push constants...).
- Tests find fixtures via the `XPOSE_KERNEL_DIR` compile definition
  (absolute path — binary is not relocatable; adjust for Android pushes).

## Device portability & memory coherency (READ before touching Buffer/transfers)
- **Desktop-green != device-green.** Every desktop driver (RADV/NVIDIA/llvmpipe)
  hands back HOST_VISIBLE **+ HOST_COHERENT** memory, so the non-coherent path is
  NEVER exercised on desktop. Android GPUs (Mali-G57 verified) expose
  HOST_VISIBLE | **HOST_CACHED (non-coherent)** memory — a real, correct memory
  type, not a bug (cached = faster CPU access). This bit us on the first real
  device run: 6 buffer-transfer tests passed on all 3 desktop drivers, failed on Mali.
- **Non-coherent = manage the CPU cache, NOT copy data.** UMA devices share one
  physical RAM (no PCIe, no staging copy needed); the only hazard is the CPU cache
  being out of sync with RAM. After a host WRITE to mapped memory ->
  `vmaFlushAllocation` (push cache to RAM so the GPU sees it); before a host READ of
  GPU output -> `vmaInvalidateAllocation` (drop stale cache, re-read RAM). Both are
  VMA no-ops on coherent memory, so they cost nothing on desktop.
- **Buffer transfer strategy** (`src/buffer.cpp`) is keyed on *mappability*, not
  coherency: direct map+memcpy (+ flush/invalidate when `!coherent_`) whenever the
  allocation is HOST_VISIBLE; the staging-copy path is reserved for truly
  non-host-visible device-local memory (discrete GPUs). `Buffer::host_visible()`
  means "directly mappable" — true for any HOST_VISIBLE allocation, coherent or not.
- **VMA flush/invalidate require the range be currently host-mapped**
  (VUID-VkMappedMemoryRange-memory-00684): map first, THEN flush/invalidate while
  mapped. Getting the order wrong null-derefs inside the Mali driver; desktop hides
  it because invalidate is a no-op on coherent memory. Full write-up:
  `agent-docs/mali-coherency-fix.md`.

## Layout
- `include/xpose/` public headers (`xpose.hpp` umbrella), `src/` impl, `tests/` googletest.
- Modules: Context/Buffer/Sync (core) + Program (SPIR-V load, ClspvReflection
  parse, pipelines) + launch() (variadic dispatch, returns Fence).
- Device-level Vulkan calls go through `Context::table()` (per-context `VolkDeviceTable`),
  never global function pointers — multiple Contexts on different drivers must coexist.
- Error strategy: exceptions (`xpose::Error`), every Vk call wrapped in `XP_CHECK`.
- launch() recycles command buffers/descriptor sets via the Fence completion
  hook (`Context::submit(cb, on_complete)`); descriptor pools grow on demand.
- Grid = GLOBAL thread counts (CUDA-style), rounded up to whole workgroups;
  default workgroup size 64x1x1 via spec constants — kernels must bound-check.


## Commit Authorship (human is sole author)

Commits and PRs must be authored solely by the human developer. Do NOT add a
⁠ Co-authored-by: Claude <noreply@anthropic.com> ⁠ trailer (or any Claude / AI
co-author trailer) to commit messages or PR bodies. This overrides any default
harness convention that would add one. The developer must be the only
contributor shown on the commit.

---
> Source: [badnikhil/Xpose](https://github.com/badnikhil/Xpose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
