## xclaw

> 纯视觉、纯键鼠的 Windows CUDA 桌面代理框架。模拟真人使用电脑的完整认知回路：

# X-Claw

纯视觉、纯键鼠的 Windows CUDA 桌面代理框架。模拟真人使用电脑的完整认知回路：

**截屏（眼睛）→ YOLO + RapidOCR + Florence-2 感知（视觉皮层）→ 结构化 JSON（语言区）→ OS 原生键鼠操作（手）**

感知层将屏幕像素转化为带编号的元素列表（纯文本 JSON），外部 Agent 的 LLM 仅消费该文本做决策，不接触任何图像数据。

仅支持 Windows (CUDA NVIDIA GPU)。

## 核心约束（不可违背）

X-Claw 是一个数字人的感官和手脚。以下约束定义了项目的身份边界，任何违反均视为架构缺陷：

1. **唯一信息源是屏幕像素。** 不读取 DOM、不调用无障碍树（Accessibility API）、不使用浏览器 DevTools / CDP、不做进程间内存读取。一切状态信息必须从截屏中重建。

2. **唯一输出通道是 OS 原生键鼠事件。** Windows 通过 ctypes SendInput。不注入 JavaScript、不调用应用程序 API。复制粘贴（Ctrl+C/V）属于真人正常操作，Agent 可以使用；但禁止将剪贴板作为绕过键鼠的隐蔽数据传输通道。

3. **感知层输出纯文本，LLM 不接触图像。** 感知管线的最终产物是结构化 JSON（元素 ID、bbox、类型、文本内容），外部 Agent 的 LLM 仅基于该文本做推理和决策。不向 LLM 传递截图、不依赖多模态能力、不引入"让 LLM 看一眼图片辅助判断"的回退路径。视觉理解的全部责任由感知层承担。

4. **服务单个 Agent。** 架构为单 Agent 独占设计，不考虑多 Agent 并行、分布式调度、远程 RPC。

5. **感知层与执行层通过协议解耦，但不引入中间层语义推理。** 感知层输出带编号标注的元素列表，所有高层语义理解（意图识别、任务规划）由外部 LLM 在纯文本域完成。禁止在管线中插入"UI 语义分类器"、"控件类型推断器"、"意图预测器"等中间智能层。

6. **反检测是一等需求。** HumanizeStrategy 不是可选的锦上添花，而是与感知、执行同等重要的核心能力。任何新增的操作路径都必须经过人性化策略层，不允许存在绕过 HumanizeStrategy 的快捷通道。

## 开发环境

- Python 3.12（通过 `.python-version` 锁定）
- 包管理：[uv](https://docs.astral.sh/uv/)
- 安装依赖：`uv sync`（PyTorch 从 `download.pytorch.org/whl/cu121` 安装）
- 运行命令：`uv run xclaw <command>`

## 全局安装

在项目目录下执行：
```bash
uv tool install --editable . --python 3.12
```
安装后可在任意路径使用 `xclaw` 命令。卸载：`uv tool uninstall xclaw`。

## 关键路径

```
xclaw/
├── __init__.py            # 包入口
├── config.py              # 全局配置（路径、感知参数、人性化参数）
├── cli/                   # Click CLI 子包
│   ├── __init__.py        # cli group 入口 + 命令注册
│   ├── core.py            # setup() + output() + IS_DEBUG
│   ├── _silence.py        # 第三方库静默逻辑
│   └── commands/
│       ├── __init__.py
│       ├── look.py        # look 命令（直接调用 PerceptionEngine）
│       └── action.py      # click/type/press/scroll/wait 命令
├── platform/
│   ├── __init__.py        # 导出 PLATFORM / PERCEPTION_CONFIG 单例
│   ├── detect.py          # 平台检测（内存、GPU — Windows only）
│   └── gpu.py             # 感知引擎硬件配置（CUDA + TRT / CPU）
├── core/
│   ├── __init__.py
│   ├── screen.py          # 截屏（mss）
│   ├── cache.py           # LRU 感知缓存
│   ├── pipeline.py        # 视觉管线：感知 + 坐标排序
│   └── perception/
│   │   ├── __init__.py
│   │   ├── backend.py     # PerceptionBackend 协议（抽象接口）
│   │   ├── engine.py      # PerceptionEngine 单例：委托 backend 编排
│   │   ├── pipeline_backend.py # 默认后端：YOLO + OCR + Florence-2
│   │   ├── omniparser.py  # OmniDetector（YOLO 双后端：ONNX / ultralytics）
│   │   ├── icon_classifier.py # Florence-2 图标描述（OmniParser V2 fine-tuned）
│   │   ├── ocr.py         # RapidOCR (PPOCRv5 mobile ONNX, CUDA)
│   │   ├── merger.py      # IoU 去重 + YOLO/OCR 空间融合
│   │   └── types.py       # RawElement / TextBox 数据类型
├── action/
│   ├── __init__.py        # ActionBackend 单例（get_backend/set_backend）+ 向后兼容函数
│   ├── backend.py         # ActionBackend 协议（抽象接口）
│   ├── native_backend.py  # NativeActionBackend：委托 win32 模块 + HumanizeStrategy
│   ├── dry_run_backend.py # DryRunBackend：记录动作不触发 OS 事件（测试用）
│   ├── humanize_strategy.py # HumanizeStrategy 协议 + NoopStrategy / BezierStrategy
│   ├── humanize.py        # bezier_point() 纯数学工具函数
│   ├── mouse.py           # 高层点击/滚动/拖拽接口（委托 ActionBackend）
│   ├── keyboard.py        # 高层打字/按键/组合键接口（委托 ActionBackend）
│   ├── mouse_win32.py     # Windows: ctypes SendInput 鼠标（raw 操作 + 机械延迟）
│   └── keyboard_win32.py  # Windows: ctypes SendInput 键盘（raw 操作 + 机械延迟）
├── installer/
│   ├── __init__.py        # 安装工具包
│   ├── postinstall.py     # 安装后初始化（目录结构 + 模型下载 + init）
│   └── download_gui.py    # Tkinter 模型下载 GUI
├── debug/
│   ├── __init__.py
│   └── pipeline.py        # 调试用管线可视化
└── skills/
    └── SKILL.md           # Claude Code 技能入口
scripts/
├── download_models.py     # 模型下载（OmniParser V2）
├── export_yolo_onnx.py    # YOLO .pt → .onnx 导出
├── build_installer.py     # 安装包构建
└── installer.iss          # Windows Inno Setup 脚本
```

## 硬件配置

| 组件 | CUDA 模式 | CPU 回退 |
|------|-----------|----------|
| 键鼠控制 | ctypes `SendInput` | ctypes `SendInput` |
| YOLO 检测 | TensorRT EP / CUDA EP / ultralytics | CPU EP / ultralytics |
| OCR | RapidOCR PPOCRv5 mobile CUDA | RapidOCR PPOCRv5 mobile CPU |
| 图标描述 | Florence-2 FP16 CUDA | Florence-2 FP32 CPU |
| torch 索引 | `pytorch-cu121` | `pytorch-cu121` |

- `xclaw/platform/detect.py` 检测内存和 GPU
- `xclaw/platform/gpu.py` 根据 GPU 生成 `PerceptionConfig`（含 TRT 开关）
- `xclaw/action/__init__.py` 通过 `ActionBackend` 单例路由，`set_backend()` 可注入自定义后端

### PyTorch 包索引

项目通过 `[tool.uv.index]` + `[tool.uv.sources]` 配置了自定义索引：

- **pytorch-cu121** — PyTorch CUDA 12.1 wheels

OCR 使用 RapidOCR（PaddleOCR ONNX 封装），通过已有的 `onnxruntime-gpu` 走 `CUDAExecutionProvider`，无需 `paddlepaddle-gpu`。

## 环境变量

| 变量 | 说明 | 默认 |
|------|------|------|
| `XCLAW_HUMANIZE` | 设为 `1` 启用人性化鼠标移动和打字延迟 | `0` |
| `XCLAW_HOME` | 项目根目录路径（仅非 editable 安装时需要） | 自动推算 |
| `XCLAW_DATA` | 用户可写数据目录（截图、日志、模型） | 开发模式=PROJECT_ROOT，安装模式=`%LOCALAPPDATA%\X-Claw` |
| `XCLAW_TRT` | 设为 `0` 禁用 TensorRT EP（回退 CUDA EP） | `1` |
| `DEBUG` | 设为 `1` 启用 CLI 调试日志输出到 stderr | `0` |

## 感知引擎

- `PerceptionEngine`（`perception/engine.py`）是感知层单例，委托 `PerceptionBackend` 协议：
  - 默认后端 `PipelineBackend`（`pipeline_backend.py`）组合三个子模块：
    - `OmniDetector`：YOLO icon_detect，优先 TensorRT EP，回退 CUDA EP，再回退 ultralytics
    - `OCREngine`：RapidOCR PPOCRv5 mobile 中英双语 (CUDA)
    - `IconClassifier`：Florence-2 图标描述（OmniParser V2 fine-tuned Florence-2-base）
  - 可通过 `PerceptionEngine(backend=custom)` 注入自定义后端
- 每次 CLI 调用都是独立进程，每次 `look` 直接执行完整感知管线
- 模型目录搜索顺序：`PROJECT_ROOT/models` → `weights/` → `DATA_DIR/models` → `~/.xclaw/models/`
- 模型下载 + ONNX 导出：`uv run python scripts/download_models.py`
- 单独重新导出 ONNX：`uv run python scripts/export_yolo_onnx.py`

## CLI 架构

CLI 采用子包脚手架模式（`xclaw/cli/`），外部 runner 通过命令行调用：

- `xclaw look` — 截屏 + 完整感知，输出 JSON 到 stdout
- `xclaw click X Y [--double] [--button left|right|middle]` — 点击 + 自动感知
- `xclaw type TEXT` — 打字 + 自动感知（支持 stdin 管道输入 UTF-8 文本）
- `xclaw press KEY` — 按键 + 自动感知
- `xclaw hotkey COMBO` — 组合键（ctrl+c, alt+f4 等）+ 自动感知
- `xclaw scroll up|down|left|right AMOUNT` — 滚动 + 自动感知
- `xclaw drag X1 Y1 X2 Y2 [--button]` — 拖拽 + 自动感知
- `xclaw move X Y` — 移动光标（触发 hover 效果）+ 自动感知
- `xclaw hold left|right|middle down|up [--x --y]` — 按住/释放鼠标键 + 自动感知
- `xclaw cursor` — 查询光标位置和屏幕尺寸（纯查询，不触发感知）
- `xclaw wait SECONDS` — 等待 + 自动感知

所有命令输出干净 JSON 到 stdout（供 LLM 消费），DEBUG=1 时日志输出到 stderr。

## 执行层架构

- `ActionBackend` 协议（`action/backend.py`）定义 click/type/scroll/hotkey 等操作接口
- `NativeActionBackend`（`action/native_backend.py`）：默认实现，委托 win32 模块 + `HumanizeStrategy`
- `DryRunBackend`（`action/dry_run_backend.py`）：测试用，记录动作不触发 OS 事件
- `HumanizeStrategy` 协议（`action/humanize_strategy.py`）：
  - `NoopStrategy`：无人性化，直接操作
  - `BezierStrategy`：贝塞尔曲线移动 + 抖动 + 随机延迟
- `XCLAW_HUMANIZE=1` 时自动使用 `BezierStrategy`，否则用 `NoopStrategy`
- `set_backend(DryRunBackend())` 可在测试中替换整个执行层

## 模型说明

- OmniParser 源码位于 `OmniParser/` 目录
- YOLO icon_detect 模型：`models/icon_detect/model.onnx`（首选）或 `model.pt`
- Florence-2 图标描述：`models/icon_caption_florence/`（OmniParser V2 fine-tuned）
- 模型存放在 `models/`（首选）或 `weights/`（向后兼容）目录

## 配置

所有可配置项集中在 `xclaw/config.py`，不要在其他模块中硬编码路径或参数。

## 测试

```bash
uv run pytest                        # 跑默认测试（排除 gpu/bench）
uv run pytest -m gpu                 # 需要 GPU + 模型
```

## 安装包构建

```bash
# Windows (需要 Inno Setup)
python scripts/build_installer.py --platform windows  # → dist/XClaw-x.x.x-Setup.exe
```

### 路径架构（安装模式 vs 开发模式）

| 路径 | 开发模式 | 安装模式 |
|------|---------|---------|
| `PROJECT_ROOT` | 项目根目录 | 安装目录 |
| `DATA_DIR` | = PROJECT_ROOT | `%LOCALAPPDATA%\X-Claw` |
| `MODELS_DIR` | `PROJECT_ROOT/models` | `DATA_DIR/models` |
| `SCREENSHOTS_DIR` | `PROJECT_ROOT/screenshots` | `DATA_DIR/screenshots` |

### 用户安装流程

1. 双击安装包（.exe）
2. 首次启动自动执行 `uv sync` 安装 Python + 依赖（~1.5GB）
3. Tkinter GUI 下载模型（~500MB）
4. `xclaw look` 验证感知管线正常工作
5. 完成

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SoberPizza) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
