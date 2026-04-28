## reshade-cv-version2-0

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a ReShade addon that extracts training data from games for deep neural networks (DNN), including RGB images, depth maps, camera matrices, and semantic segmentation. It supports creating datasets for NeRF reconstruction, SLAM, monocular depth estimation, and semantic segmentation.

The addon hooks into game rendering pipelines via ReShade 5.8.0+ to capture:
- RGB frames
- Depth buffers (calibrated physical distance when supported)
- Camera extrinsic matrices (cam2world) and FOV
- Semantic segmentation data (DirectX 10/11 only)

## 重要规则

**修改完成后自动编译部署测试。** 代码修改完成后，自动执行以下流程：
1. 调用 `msbuild gcv_reshade.sln /p:Configuration=Release /p:Platform=x64` 编译
2. 编译成功后，将 `build/x64/Release/cv_captures.addon` 复制到当前游戏的可执行文件目录（从 `doc/game/<GameName>.md` 顶部获取路径）
3. 启动游戏进行测试（通过 Steam 或直接运行 exe）
如果编译失败，立即分析错误并修复，不要等用户干预。

**PR 默认提交到当前 fork（origin）。** 创建 PR 时，除非用户明确指定提交到 upstream，否则一律使用 `--repo` 指向当前 fork（即 `origin` 对应的仓库），不要提交到 upstream。

**每次更改主框架时同步更新 changelog。** 修改框架文件后，按 `/update-changelog` skill 的规范更新 `doc/CHANGELOG.md`。

## Build Commands

### Build with Visual Studio
```bash
# Open solution in Visual Studio 2022
start gcv_reshade.sln

# Or build from command line with MSBuild
msbuild gcv_reshade.sln /p:Configuration=Release /p:Platform=x64
```

The build outputs `cv_captures.addon` to `build/x64/Debug/` or `build/x64/Release/`.

### Dependencies via vcpkg

Required vcpkg packages (target `x64-windows`):
- `eigen3` - Matrix operations
- `nlohmann-json` - JSON serialization
- `xxhash` - Fast hashing

ReShade 5.8.0 source must be placed at `..\reshade-5.8.0` (one level up from this repo).

### Installation to Game

1. Install ReShade 6.4.0+ to your game directory
2. Copy the compiled `cv_captures.addon` to the game executable directory (where `ReShade.log` is)
3. For games requiring camera matrix extraction, install the appropriate mod script from `mod_scripts/` (see game-specific instructions in script comments)
4. Copy `reshade_shaders/*.fx` to the ReShade shader search path if needed

## Architecture

### High-Level Data Flow

1. **Game Rendering** → ReShade hooks intercept draw calls and frame buffers
2. **ReShade Addon (gcv_reshade)** → Captures RGB, depth, and camera data each frame
3. **Game Interface (gcv_games)** → Game-specific logic extracts camera matrix and converts depth values
4. **Writer Threads** → Serialize data to disk asynchronously (JSON + NPY + FPZIP compressed depth)
5. **Post-processing (python_threedee)** → Convert captures to NeRF transforms.json or point clouds

### Core Components

**`gcv_reshade/` - ReShade Addon Core**
- `main.cpp`: Main addon entry point with ReShade API hooks. Handles F11 (single capture) and F9/F10 (video recording start/stop) hotkeys.
- `copy_texture_into_packedbuf.cpp/.h`: Texture copying utilities for RGB and depth buffers. Supports shader-based depth capture for DX12/Vulkan via `DepthCapture.fx`.
- `grabbers.cpp/.h`: Frame buffer grabbing logic
- `recorder.cpp/.h`: Video recording state machine
- `image_writer_thread_pool.cpp/.h`: Async disk I/O for captured frames

**`gcv_games/` - Game-Specific Implementations**
- Each game has its own class (e.g., `Cyberpunk2077.cpp`, `ResidentEvils.cpp`, `HorizonZeroDawn.cpp`)
- Game classes must implement:
  - `gamename_verbose()`: Display name
  - `camera_dll_name()` / `camera_dll_mem_start()`: Where to find camera data in memory
  - `convert_to_physical_distance_depth_u64()`: Convert raw depth buffer values to meters
  - Camera matrix extraction logic (often via scripted buffer or memory scanning)
- `game_interface_factory.cpp`: Factory pattern for auto-registering game implementations

**`gcv_utils/` - Shared Utilities**
- `camera_data_struct.cpp/.h`: `CamMatrixData` struct for extrinsic matrices and FOV, serialization to JSON
- `memread.cpp/.h`: Memory reading helpers for extracting data from game process memory
- `depth_utils.h`: Depth conversion utilities
- `scripted_cam_buf_templates.h`: Templates for extracting camera data from script-injected memory buffers

**`mod_scripts/` - Game Mod Scripts** (not part of C++ build)
- Lua/CET scripts that run inside games to export camera matrices to shared memory
- Examples: `cyberpunk2077_cyberenginetweaks_mod_init.lua`, `residentevil_read_camera_matrix_transfcoords.lua`

**`python_threedee/` - Post-Processing Scripts** (not part of C++ build)
- `convert_game_snapshot_jsons_to_nerf_transformsjson.py`: Aggregate per-frame JSONs into NeRF-ready transforms.json
- `load_point_cloud.py`: Visualize captures as 3D point clouds in Open3D, save as PLY/PCD
- `_jake/run_pipeline.py`: **主重建脚本**，从采集数据重建点云的完整流水线。F9 连续录制的丢帧处理（RGB/深度/相机矩阵帧号对齐）需要在此脚本中正确剔除

**`reshade_shaders/` - Custom ReShade Effects**
- `DepthCapture.fx`: Shader for capturing depth on DX12/Vulkan where direct buffer access isn't available
- `segmentation_visualization.fx`: Real-time visualization of semantic segmentation
- `displaycamcoords.fx`: Display camera coordinates overlay

### Depth Capture Strategies

The addon supports two depth capture methods:

1. **Direct Buffer Access** (DX10/DX11/some DX12): Read depth/stencil buffer directly from GPU memory
2. **Shader-Based Capture** (DX12/Vulkan fallback): `DepthCapture.fx` renders ReShade's depth buffer to a texture that the addon can read

The code in `gcv_reshade/main.cpp` (lines 40-85) handles automatic fallback to shader-based capture when direct access fails.

### Camera Matrix Extraction

Games use different methods to expose camera data:

- **Scripted Buffer**: Mod injects camera matrix into a known memory location (e.g., Cyberpunk 2077 via CyberEngineTweaks)
- **Memory Scanning**: Search process memory for camera matrix patterns (less reliable, game version dependent)
- **IGCS Connector**: Interface with IGCS camera tools (used by some Resident Evil games)

Game classes specify the extraction method via `camera_dll_matrix_format()` return value.

### Semantic Segmentation (DX10/DX11 only)

Uses RenderDoc code (`renderdoc/`) to intercept draw calls and assign mesh/shader IDs to semantic categories. Mapping mesh IDs to semantic labels is manual per-game effort.

## Workflow: Adding a New Game

1. Create `gcv_games/NewGame.cpp` and `gcv_games/NewGame.h` based on a similar game
2. Implement required `GameInterface` virtual methods:
   - Camera data extraction logic
   - Depth conversion formula (if known)
3. Register the game class with `REGISTER_TYPE` macro at bottom of .cpp
4. If needed, create a mod script in `mod_scripts/` to export camera matrix
5. Add the .cpp/.h to `gcv_reshade/gcv_reshade.vcxproj` and `.vcxproj.filters`
6. Build and test in-game

See `gcv_games/Cyberpunk2077.cpp` for a well-documented example using scripted camera buffer.

## 新建游戏类时必须参考的文档

在实现各方法之前，**先查阅 `doc/` 下对应的总结文档**，选取已经验证过的模板，避免重复踩坑。

### 深度转换公式（`convert_to_physical_distance_depth_u64`）

**参考**：[`doc/summary_depth_conversion.md`](doc/summary_depth_conversion.md)

- 该文档列出了全部公式类型（P/E/R/L 类）及适用条件
- 对于 D24S8 格式的游戏，额外参考 `doc/summary_depth_conversion.md` 中的"D24S8 格式游戏详解"节
- **关键判断**：depthval 是 float 位模式（memcpy 还原）还是 D24 整数（除以 16777215）？两者不可混用
- 若尚无标定数据，默认先用 P 类（透视逆变换）占位，后用 `python_threedee/fit_depth_hfw.py` 拟合
- 优先使用 `P32 / (depth - P22)` 的通用形式，不要为每个游戏写等价但风格不同的公式

**深度公式代码风格**（拟合或编写时均须遵循）：

```cpp
// 显式写出 n/f，再推导常数，不要直接写算好的数字
const float n = 0.1f;
const float f = 10000.0f;
const float numerator_constant = (-f * n) / (n - f);
const float denominator_constant = n / (n - f);
return numerator_constant / (depth - denominator_constant);
```

- 若拟合脚本输出的是 `P32 / (depth - P22)` 形式（直接数值），须将其反推出 n、f（或保留 P32/P22 变量名），再按上述模板写入代码，**不要直接硬写裸数字**
- `1-depth` 变换（reversed-Z）与 `depth` 直接使用的公式在数学上等价，但代码里要注释清楚使用哪种，参考 `doc/summary_depth_conversion.md`

### IGCS 相机处理（`process_camera_buffer_from_igcs`）

**参考**：[`doc/summary_igcs_camera_processing.md`](doc/summary_igcs_camera_processing.md)

- 该文档将 43 个游戏的实现分为 E1-E11（欧拉角输入）和 M1-M3（旋转矩阵输入）共 14 种模式
- 先确认游戏属于哪种模式，直接复用对应模板，不要从头推导
- 最常见模式：**E1**（14 个游戏），UE→CV 坐标系，列 2/3 取反
- **两个重载二选一实现即可**，不必两个都写：
  - 欧拉角版 `process_camera_buffer_from_igcs(..., float roll, float pitch, float yaw, float fov)` — 对应 E 类模板，`prefer_camera_matrix()` 返回 `false`
  - 旋转矩阵版 `process_camera_buffer_from_igcs(..., const float* camera_matrix, float fov)` — 对应 M 类模板，`prefer_camera_matrix()` 返回 `true`

### FOV 水平/垂直（`fov_is_vertical` 参数）

**参考**：[`doc/summary_fov_classification.md`](doc/summary_fov_classification.md)

- 该文档列出了所有已接入游戏的 FOV 类型（水平/垂直）
- 16:9 下：垂直 FOV 通常 40-70°，水平 FOV 通常 65-105°；游戏设置中的 FOV 值可辅助判断
- 错误标记 FOV 类型会导致单帧点云圆形变椭圆、多帧远处无法对齐

### 相机矩阵格式（`camera_dll_matrix_format` / C2W 变换）

**参考**：[`doc/summary_cam2world_matrix.md`](doc/summary_cam2world_matrix.md)

- 该文档说明了统一的 3×4 行主序格式、坐标系约定，以及各平台的转换规则

### 参考已完成游戏的文档

`doc/game/` 下每个游戏一个文档，记录了标定结论、踩坑和调试过程：

| 引擎 / 特征 | 参考文档 |
|---|---|
| Decima 引擎（HZD/HFW/DS） | `HorizonForbiddenWest.md`、`HorizonZeroDawn.md`、`DeathStranding_DirectorsCut.md` |
| CryEngine（深度拟合流程） | `Crysis2_Remastered.md`、`Crysis3_Remastered.md` |
| 内存矩阵格式分析 | `Crysis_Remastered.md` |
| 索尼 PC 端（深度缓冲尺寸问题） | `SpiderMan_MilesMorales.md`、`SpiderMan_Remastered.md`、`RatchetAndClank_RiftApart.md` |
| 内存相机读取（无 IGCS） | `GodOfWar_Ragnarok.md`、`HorizonZeroDawn.md` |
| DepthCapture.fx + memcpy float | `Cyberpunk2077.md`、`HorizonForbiddenWest.md` |

### 游戏可执行文件目录与调试文件

每个 `doc/game/*.md` 文档顶部记录了该游戏的可执行文件所在目录路径（`> **游戏可执行文件目录**：...`）。调试或编写代码时，可利用该路径访问游戏目录下的关键文件：

- `<游戏目录>/ReShade.log` — ReShade 日志，排查 addon 加载失败、hook 异常等问题
- `<游戏目录>/cv_saved/` — 采集数据输出目录，包含 JSON、深度、RGB 等文件
- `<游戏目录>/reshade-shaders/Shaders/` — ReShade shader 目录（如 DepthCapture.fx）

新建游戏文档时参考 [`doc/game/template.md`](doc/game/template.md) 模板，确保目录字段已填写。

### 接入完成后的校验流程

**参考**：[`doc/workflow.md`](doc/workflow.md)

按顺序校验：深度拟合 → 平移缩放 → FOV 水平/垂直 → 旋转矩阵方向

## Git Workflow (from BUILD.md)

**Main branch**: Verified, working commits only

**Feature branches**: For testing new features or games
```bash
git switch -c wip/gamename-feature
# Work and commit...
```

**Merge back to main**:
```bash
git switch main
git pull --rebase origin main
git merge --no-ff wip/gamename-feature
git push origin main
```

**Temporary testing** (detached HEAD):
```bash
git switch --detach <commit-hash>
# Test...
git switch main  # Discard changes and return
```

## Usage In-Game

- **F11**: Capture single frame (RGB + depth + JSON metadata)
- **F9**: Start video recording mode
- **F10**: Stop video recording mode

### F9 录制画质设置要求

F9 连续录制时，**必须关闭垂直同步（V-Sync OFF）** 并使用游戏内帧率限制。V-Sync ON 时 `Present()` 阻塞等待 vblank，导致 IGCS 相机数据超前于 GPU 帧缓冲区 1-3 帧，运动中重建会错位。关闭 V-Sync 后 `Present()` 立即返回，相机与帧内容同步。详见 `doc/game/Borderlands4.md` 中的实验数据。

Captured data goes to `cv_saved/` directory with structure:
```
cv_saved/
  <timestamp>/
    00000_meta.json      # Camera matrix, FOV, resolution
    00000_depth.fpz      # FPZIP compressed depth (or .npy)
    00000.jpg            # RGB frame (from video extraction)
```

## Python Post-Processing

After capturing frames:

1. Extract video to individual JPG frames with ffmpeg
2. Run `python_threedee/convert_game_snapshot_jsons_to_nerf_transformsjson.py` to create transforms.json
3. Or run `python_threedee/load_point_cloud.py` to visualize as 3D point cloud

## Important File Formats

- **JSON metadata**: Contains `extrinsic_cam2world` (4x4 matrix), `fov_v_degrees`, `fov_h_degrees`, image dimensions
- **Depth format**: Either `.fpz` (FPZIP lossless float compression) or `.npy` (NumPy array). Values are in meters when `can_interpret_depth_buffer()` returns true for that game.
- **Semantic segmentation**: `.png` with pixel values representing semantic class IDs

## Notes

- Not all games support all features. Check README.md compatibility tables.
- Depth buffer interpretation requires per-game calibration (see `convert_to_physical_distance_depth_u64()` implementations).
- Camera matrix extraction requires game-specific reverse engineering or mod support.
- For new game support, start with depth-only capture (works for most ReShade-compatible games), then add camera matrix support if needed.

## fit-depth

- **fit-depth** (`.claude/skills/fit-depth/SKILL.md`) - run depth calibration fitting, update game C++ code and doc. Trigger: `/fit-depth`
When the user types `/fit-depth`, invoke the Skill tool with `skill: "fit-depth"` before doing anything else.

## update-changelog

- **update-changelog** (`.claude/skills/update-changelog/SKILL.md`) - update `doc/CHANGELOG.md` after framework file changes. Trigger: `/update-changelog`
When the user types `/update-changelog`, invoke the Skill tool with `skill: "update-changelog"` before doing anything else.

## update-game-entry

- **update-game-entry** (`.claude/skills/update-game-entry/SKILL.md`) - update a game's ant_ref doc entry and installer package. Trigger: `/update-game-entry`
When the user types `/update-game-entry`, invoke the Skill tool with `skill: "update-game-entry"` before doing anything else.

## deploy-addon

- **deploy-addon** (`.claude/skills/deploy-addon/SKILL.md`) - build, deploy addon to game directory, and maintain version history in `../_game/`. Trigger: `/deploy-addon`
When the user types `/deploy-addon`, invoke the Skill tool with `skill: "deploy-addon"` before doing anything else.

## find-proj-matrix

- **find-proj-matrix** (`.claude/skills/find-proj-matrix/SKILL.md`) - find projection matrix in game memory, implement dynamic near-plane or FOV reading. Trigger: `/find-proj-matrix`
When the user types `/find-proj-matrix`, invoke the Skill tool with `skill: "find-proj-matrix"` before doing anything else.

## evaluate-reconstruction

- **evaluate-reconstruction** (`.claude/skills/evaluate-reconstruction/SKILL.md`) - evaluate point cloud reconstruction quality: differential thickness + TSDF residual + vision analysis → structured JSON feedback. Trigger: `/evaluate-reconstruction`
When the user types `/evaluate-reconstruction`, invoke the Skill tool with `skill: "evaluate-reconstruction"` before doing anything else.

## debug-camera

- **debug-camera** (`.claude/skills/debug-camera/SKILL.md`) - automated 6-frame camera debugging: score cam2world geometry, search column/worldbasis variants, fix game class. Trigger: `/debug-camera`
When the user types `/debug-camera`, invoke the Skill tool with `skill: "debug-camera"` before doing anything else.

## graphify

This project has a graphify knowledge graph at graphify-out/.

Rules:
- Before answering architecture or codebase questions, read graphify-out/GRAPH_REPORT.md for god nodes and community structure
- If graphify-out/wiki/index.md exists, navigate it instead of reading raw files
- After modifying code files in this session, run `python3 -c "from graphify.watch import _rebuild_code; from pathlib import Path; _rebuild_code(Path('.'))"` to keep the graph current

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zikun-dai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
