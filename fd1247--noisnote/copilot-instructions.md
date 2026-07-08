## noisnote

> This file provides guidance to Codex when working in this repository.

# AGENTS.md

This file provides guidance to Codex when working in this repository.

## 项目概述

音频转录与总结工具是一个面向 Windows 的 PySide6 / Qt Widgets 桌面应用。用于录制系统音频、录制麦克风或导入本地音视频文件，将输入保存为历史记录，再通过本地 ASR 模型转录文字，并可调用 OpenAI 兼容的 LLM API 生成总结。

```text
创建录音或导入音视频 -> 保存历史记录 -> ASR 转录（含逐句时间轴） -> 可选 LLM 总结 -> 音频回放 / 导出 -> 查看结果
```

## 技术栈

|   层级   |                        技术                        |
| -------- | -------------------------------------------------- |
| GUI      | PySide6 / Qt Widgets                               |
| 系统音频 | SoundCard (WASAPI Loopback)                        |
| 音频回放 | QMediaPlayer / QAudioOutput                        |
| ASR 推理 | Qwen3-ASR GGUF（ONNX + llama.cpp），本地模型推理     |
| 时间戳   | Qwen3-ForceAligner GGUF（ONNX + llama.cpp）         |
| LLM 总结 | OpenAI 兼容 API / Anthropic 兼容 API                |
| 音频处理 | ffmpeg / ffprobe                                   |
| 模型下载 | ModelScope（优先）+ GitHub（备用）                  |
| 日志     | JSON Lines，写入 `~/Documents/NoisNote/logs/` |
| 测试     | pytest，Qt 测试使用 `QT_QPA_PLATFORM=offscreen`     |
| 打包     | PyInstaller（onedir 模式）                          |

## 常用命令

```bash
# 启动应用
python main.py

# 运行常规单元测试
python -m pytest tests/test_qt_history.py tests/test_qt_models_gguf.py tests/test_qt_model_workers.py -q

# 运行扩展单元测试（含回放、时间轴、导出、对话框、预处理、转录子进程）
python -m pytest tests/test_qt_main_window_p0.py tests/test_qt_main_window_ch07.py tests/test_qt_dialogs.py tests/test_qt_history_widgets.py tests/test_timestamp_alignment_app.py tests/test_audio_preprocess.py tests/test_transcription_worker.py -q

# 运行 WASAPI 录音手动测试（需要 Windows 音频设备）
python tests/test_wasapi_record.py

# 安装依赖
pip install -r requirements.txt
```

## 项目结构

```text
main.py                                  # 应用入口
src/
  __init__.py                           # 包版本信息
  app/                                  # 应用核心
    application.py                      # QApplication 初始化、日志、异常钩子
    main_window.py                      # 主窗口骨架、UI 布局、Mixin 组装
    config.py                           # 默认配置、配置读写、模型清单
    version.py                          # 语义化版本号
    update.py                           # GitHub Releases 更新检查
    diagnostics.py                      # ASR 运行时诊断工具
  audio/                                # 音频录制模块
    recorder.py                         # AudioRecorder 录制引擎
    device_manager.py                   # DeviceManager WASAPI 设备管理
    types.py                            # CaptureMode、CaptureSettings 等数据类
    preprocess.py                       # 音视频探测、格式转换
  asr/                                  # ASR 转录引擎
    engine.py                           # TranscriptionEngine 高层封装
    runtime.py                          # Qwen3AsrGgufRuntime vendor 封装
    types.py                            # 数据模型（含 TimelineSegment/Token、进度、设备解析）
    timestamps.py                       # 时间轴生成、HTML/SRT 导出、时间格式化
    worker_process.py                   # ASR 子进程入口，通过 JSON Lines stdout 通信
    utils.py                            # 转录文本清理
  llm/                                  # LLM 总结服务
    summarizer.py                       # Summarizer（system/user 角色分离的 prompt 格式）
  history/                              # 历史记录管理
    service.py                          # HistoryService CRUD、时间轴读写、ASR 元数据
    storage.py                          # 文件存储和元数据管理
    types.py                            # HistoryRecord、HistoryStatus
  model_registry/                       # 模型下载管理
    service.py                          # ModelService 模型目录与校验
    downloader.py                       # ModelDownloadManager 任务生命周期
    worker.py                           # ModelDownloadWorker 下载线程
    download.py                         # 下载源选择、文件下载、解压、校验
    types.py                            # ModelStatus、ModelCatalogEntry 等
  handlers/                             # MainWindow Mixin（每个文件对应一个功能域）
    media_import.py                     # 文件导入和拖拽
    remote_import.py                    # 远程公开视频链接导入（yt-dlp、字幕优先、音频降级）
    recording.py                        # 录音设备和流程
    processing.py                       # 处理状态和结果保存
    transcription.py                    # ASR 转录和重新转录
    summary.py                          # LLM 总结
    settings.py                         # 设置导航和配置保存
    history_view.py                     # 历史列表搜索、筛选、选择、右键菜单、详情加载
    timeline_view.py                    # 逐句时间轴展示、回放位置高亮、复制/格式化
    playback.py                         # 音频回放、进度条、倍速、快捷键
    export.py                           # 导出入口（转录/timeline/总结 -> txt/srt/markdown）
  workers/                              # 后台线程
    transcription.py                    # TranscriptionWorker（子进程 ASR）
    summary.py                          # SummaryWorker
    preprocess.py                       # AudioPreprocessWorker
    remote_import.py                    # RemoteProbeWorker / RemoteImportWorker
  remote_import/                        # 远程链接导入服务
    ytdlp_client.py                     # yt-dlp 适配、cookies、站点错误归类
    service.py                          # 字幕下载/解析，失败后音频降级
    subtitles.py                        # SRT/VTT/常见 JSON 字幕转换
    errors.py                           # 远程导入错误类型与用户提示
    types.py                            # RemoteMediaInfo / RemoteImportOptions 等数据结构
  ui/                                   # Qt 界面组件
    styles.py                           # 全局 Qt 样式表（含回放栏、时间轴、设置、对话框等样式）
    icons.py                            # SVG 图标加载（action/make_icon、eye、combo_arrow 等）
    sidebar.py                          # 侧边栏构建（主侧栏 + 设置侧栏）
    recording.py                        # 录音页面
    content.py                          # 历史详情页（含 SeekSlider、PlaybackRateCombo、回放栏、标签切换、导出菜单）
    result.py                           # 结果状态辅助（转录/总结文本设置、标签切换）
    settings.py                         # 设置面板（通用/模型/热词/快捷键）
    model_panel.py                      # 模型管理子页面（树形模型浏览器）
    widgets/                            # 可复用组件
      history_item.py                   # 历史记录列表项（ElidedLabel + 右键菜单）
      dialogs.py                        # 对话框系统（确认/警告/输入/选择 + 键盘导航）
      update_dialog.py                  # 版本更新对话框
  utils/                                # 通用工具
    logging.py                          # JSON Lines 日志初始化、脱敏
    ffmpeg.py                           # ffmpeg/ffprobe 发现
  hotwords/                             # 热词管理
    service.py                          # HotwordService 热词增删改查、激活控制
    types.py                            # HotwordSet 数据模型和验证常量
    import_export.py                    # 热词表 JSON 导入导出
vendor/qwen3-asr-gguf/                  # Qwen3-ASR-GGUF 第三方 Python 源码
  qwen_asr_gguf/inference/              # ASR 推理源码（上游 e790e3b + 3 处定制）
  qwen_asr_gguf/inference/bin/          # llama.cpp DLL（.gitignored, 由 download_deps.py 下载）
docs/                                   # 文档
  modules/                              # 各模块架构文档
  roadmap/                              # 路线图
  dev/                                  # 开发阶段文档
tests/                                  # 单元测试
scripts/                                # 构建和发布脚本
```

## 核心设计

### 数据目录

配置文件存储在系统 AppData 目录，用户数据存储在 Documents：

```text
%APPDATA%/NoisNote/
  config.json

~/Documents/NoisNote/
  data/                                  # 历史记录
  models/                                # ASR 模型
  logs/                                  # 日志
```

历史记录采用"一条记录一个文件夹"的结构。每条记录目录包含：
- `audio.wav` — 录音/导入的音频文件
- `metadata.json` — 元数据（含 ASR 诊断、时间戳配置等）
- `transcript.txt` — 转录文本
- `summary.md` — LLM 总结（Markdown）
- `timeline.json` — 逐句时间轴结构化数据
- `external_subtitle.srt` — 远程链接外部字幕（按需生成）
- `transcript.srt` — SRT 字幕导出（按需生成）

导入音频默认记录源文件路径，不复制源文件。导入视频在开始转录时才提取音轨。远程链接导入优先使用外部字幕生成转录和时间轴，字幕不可用或处理失败时再下载音频并标准化为 `audio.wav`。下载中的模型先写入 `.download-<name>` 临时目录，校验通过后再移动到最终目录。

### 配置结构

主要配置段：

- `selected_asr`：ASR 模型名、模型路径、推理设备
- `qwen3_asr_gguf`：GGUF runtime 工具目录、chunk、上下文、`enable_timestamps` 等运行参数
- `llm`：API Key、模型名、Base URL、供应商（openai / anthropic）
- `audio`：录音模式、设备选择、`auto_transcribe`/`auto_summarize` 开关（默认均为 `false`）、预处理参数
- `models`：已下载模型记录
- `hotword_sets` / `active_hotword_set_ids`：热词表及激活状态

`config.py` 负责补齐新增默认字段，并通过 `_normalize_model_config` 迁移旧模型名和设备值。

### MainWindow Mixin 架构

MainWindow 通过 Python 多重继承组装功能，每个 Mixin 对应 `handlers/` 目录下的一个文件。当前共 11 个 Mixin：

| Mixin | 文件 | 职责 |
| ----- | ---- | ---- |
| ImportHandlers | media_import.py | 文件导入/拖拽、导入后自动转录 |
| RemoteImportHandlers | remote_import.py | 远程链接解析、字幕/音频导入 |
| RecordingHandlers | recording.py | 录音设备、录音流程、录音页状态 |
| ProcessingHandlers | processing.py | 共享处理状态、结果保存、worker 清理 |
| TranscriptionHandlers | transcription.py | ASR 转录生命周期、重新转录、时间轴保存 |
| SummaryHandlers | summary.py | LLM 总结生命周期、手动总结 |
| SettingsHandlers | settings.py | 设置导航、配置持久化、模型变更回调 |
| HistoryViewHandlers | history_view.py | 历史列表搜索/筛选、选择、详情加载、右键菜单 |
| TimelineViewHandlers | timeline_view.py | 逐句时间轴渲染、回放位置高亮 |
| PlaybackHandlers | playback.py | QMediaPlayer 音频回放、进度/倍速、快捷键 |
| ExportHandlers | export.py | 导出 txt/srt/markdown |

Mixin 之间不应直接互相调用，公共逻辑提取到 MainWindow 自身。MainWindow 自身负责 `__init__` 中的状态初始化、`_build_*` UI 构建方法和少量跨域方法（如 `copy_panel_text`、`delete_current_record`）。

### 后台任务

耗时任务不能在 UI 线程中执行：

- 转录：`TranscriptionWorker` (QThread + subprocess)。通过 `subprocess.Popen` 启动
  `src/asr/worker_process.py` 独立子进程执行 ASR 推理，主进程与子进程通过 JSON Lines
  stdout 通信。子进程隔离可防止 native crash 带走主界面。
- 总结：`SummaryWorker` (QThread)
- 音频预处理：`AudioPreprocessWorker` (QThread)
- 远程链接：`RemoteProbeWorker` (QThread) 解析 yt-dlp 元数据，`RemoteImportWorker`
  (QThread) 下载字幕或音频并写入历史记录。网络访问、yt-dlp、字幕下载和音频转码都不能在 UI 线程中执行。
- 模型下载：`ModelDownloadWorker`，生命周期由 `ModelDownloadManager` 管理

后台线程不得直接操作 Qt 控件，只能通过 signal/slot 通知 UI。

### ASR 子进程架构

`TranscriptionWorker` 通过 `subprocess.Popen` 启动独立的 ASR 子进程（入口为
`src/asr/worker_process.py`），而非直接在 QThread 中调用 vendor 推理代码：

- **主进程**（`TranscriptionWorker`）：管理子进程生命周期，通过 JSON Lines stdout
  接收进度/完成/失败消息，转换为 Qt signal 发送给 UI。
- **子进程**（`worker_process.py`）：加载模型并执行真实 ASR 推理，通过 `_emit()`
  输出 JSON Lines 消息到 stdout。不依赖任何 Qt 模块。
- **优势**：native crash（如 llama.cpp 段错误）不会直接带走主界面进程，用户可
  看到失败提示后重试。
- **通信协议**：每行一条 JSON，`kind` 字段区分 `transcription_progress` / `completed` /
  `failed`，stderr 用于诊断日志。

### 音频回放

录音或导入的音频支持在历史详情页内回放：

- 使用 `QMediaPlayer` + `QAudioOutput` 播放
- 回放控制：播放/暂停、快退 15s、快进 15s、拖动进度条定位
- 倍速播放：0.5x / 0.75x / 1.0x / 1.25x / 1.5x / 2.0x
- 快捷键：Space（播放/暂停）、←（后退 15s）、→（前进 15s）
- 回放状态与当前选中记录绑定，切换记录时自动停止

### 逐句时间轴

转录时可选择启用时间戳功能，生成逐句时间轴：

- ASR 引擎完成转录后，通过 Qwen3-ForceAligner 对每个词进行时间对齐
- `asr/timestamps.py` 将对齐结果合并为句子级 `TimelineSegment`，存储在 `timeline.json`
- 回放时根据当前播放位置高亮对应句子（HTML 渲染，黄色背景）
- 支持导出为 SRT 字幕格式
- 可在设置中开关（默认关闭），需要下载 ForceAligner 辅助模型

### LLM 总结 Prompt 格式

`Summarizer` 使用标准的 system/user 角色分离格式：

- **OpenAI 兼容 API**：`system` 角色设定助手身份，`user` 角色携带转录文本
- **Anthropic API**：顶层 `system` 参数设定助手身份，`messages[0]` 为 `user` 角色

不再使用将指令和文本混在一条 user 消息中的旧格式。

### Vendor 依赖

Qwen3-ASR-GGUF 推理引擎以源码形式集成在 `vendor/qwen3-asr-gguf/` 中，运行时通过 `sys.path.insert` + `ctypes.CDLL` 动态加载。llama.cpp DLL 来自官方预编译的 `llama-b7798-bin-win-vulkan-x64.zip`，由 `scripts/download_deps.py` 下载到 `vendor/qwen3-asr-gguf/qwen_asr_gguf/inference/bin/`（该目录已 .gitignore）。项目对上游源码做了 3 处定制，详见 `vendor/qwen3-asr-gguf/VENDOR_SOURCE.md`。

### 模型管理

正式模型清单包含：

| 模型 | 类型 | 用途 |
| ---- | ---- | ---- |
| Qwen3-ASR-0.6B-GGUF | ASR | 轻量版，适合日常录音 |
| Qwen3-ASR-1.7B-GGUF | ASR | 高精度版 |
| Qwen3-ForceAligner-0.6B-GGUF | 辅助 | 时间戳对齐，按需下载 |

以 `name` 作为主键，`alias` 仅用于兼容旧配置。新增模型需同步 `app/config.py`、`asr/runtime.py`、`model_registry/service.py` 和测试文件。

## 当前约束

- 目标运行环境是 Windows 10/11。录音模块依赖 WASAPI Loopback，不可移植到其他平台。
- 默认设备策略偏保守，`auto` 映射到 CPU。
- GPU 推理路径仅实现 DirectML（Windows），无 CUDA/CoreML 适配。
- 仅识别 `~/Documents/NoisNote/models/` 下的模型目录。
- 快捷键页面是预留入口，全局快捷键体系尚未完成（仅回放快捷键已实现）。
- 回放功能依赖 `QMediaPlayer`，支持的音频格式取决于系统解码器。

## 开发规范

- 注释使用中文，文档使用中文，编码 UTF-8。
- 不要直接改动用户本地数据目录，除非用户明确要求。
- 不要提交 API Key、模型文件、录音文件、cookies、缓存和本地 IDE 配置。远程导入 cookies 应放在用户数据目录，不应放在仓库内。
- 变更历史记录、模型管理或配置结构时，需补充对应单元测试。

## 验证建议

代码改动后优先运行：

```bash
python -m pytest tests/test_qt_history.py tests/test_qt_models_gguf.py tests/test_qt_model_workers.py -q
```

远程链接导入相关改动建议额外运行：

```bash
python -m pytest tests/test_qt_remote_import.py tests/test_remote_import_service.py tests/test_remote_import_subtitles.py tests/test_remote_import_ytdlp_client.py -q
```

默认不要在自动化验证中运行真实模型推理（Qwen3-ASR、CUDA、长音频转录、模型下载）。这些任务耗时长，应在用户终端手动执行。Codex 可协助分析 stdout、结果文件和日志，但不通过前台管道直接运行。

以下测试依赖本机设备或网络，默认只作为手动验收：

```bash
python tests/test_wasapi_record.py
```

---
> Source: [fd1247/NoisNote](https://github.com/fd1247/NoisNote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
