## youtube

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

YTSubViewer 是一个本地优先的 YouTube 长视频字幕翻译工具（英文 → 中文简体）。用户粘贴 YouTube 链接后，自动下载、转写、翻译并输出双语字幕和硬字幕视频。

## 启动与开发命令

```bash
# 激活虚拟环境
.venv\Scripts\activate

# 安装依赖
pip install -r requirements.txt

# 启动开发服务器（自动选端口并打开浏览器）
python app.py

# 构建便携版（PyInstaller）
powershell -File build_portable.ps1
```

## 架构概览

### 入口与服务层

- `app.py` — 应用入口，启动 uvicorn 服务器并自动打开浏览器
- `src/ytsubviewer/webapp.py` — FastAPI Web API，所有前端交互的后端接口
- `src/ytsubviewer/ui.py` — 遗留 Gradio UI（挂载在 `/legacy` 路径）

### 核心处理流程

`pipeline.py` 中的 `SubtitlePipeline` 是整个翻译流程的编排器：

1. **分析** — `YouTubeService.extract_metadata()` 通过 yt-dlp 获取视频元数据
2. **下载** — `YouTubeService.download_video()` 下载视频到 `workspace/jobs/{video_id}_{slug}/`
3. **字幕来源** — 优先级：人工英文字幕 > YouTube 自动字幕 > 本地 Whisper 转写
4. **翻译** — `DeepSeekTranslator` 调用 DeepSeek API 批量翻译字幕段
5. **后处理** — 断句优化、双语字幕生成、质量报告
6. **导出** — `VideoExportService` 使用 ffmpeg 将 ASS 字幕烧录进视频

### 后台任务系统

`background_jobs.py` 中的 `BackgroundGenerationManager` 基于 threading 实现单机后台任务队列：
- 任务状态持久化在 `{data_root}/.runtime/tasks/*.json`
- 支持取消、重试、批量任务
- 任务完成后自动调度队列中的下一个任务

### 数据模型

`models.py` 中的关键数据结构：
- `SubtitleCue` — 单条字幕（id, start, end, source_text, target_text）
- `VideoMetadata` — 视频元数据（video_id, title, duration, 字幕可用性）
- `JobArtifacts` — 任务产出物路径集合（视频、字幕、质量报告等）
- `TranslationControlConfig` — 翻译控制配置（风格预设、术语表、保护词）

### 配置体系

`config.py` 中的 `Settings` dataclass 管理所有配置，优先级：
1. 环境变量 / `.env` 文件
2. 用户设置文件（`%LOCALAPPDATA%/YTSubViewer/settings.json`）
3. 默认值

外部工具路径（ffmpeg, mpv）自动从 `.tools/` 目录搜索。

### 服务层 (`services/`)

- `youtube.py` — yt-dlp 封装，视频/字幕下载
- `transcribe.py` — faster-whisper 本地转写，自动处理 CUDA DLL 路径
- `translate.py` — DeepSeek API 翻译，支持术语表、风格预设、批量并行、质量自检与自动重译
- `export.py` — ffmpeg 字幕烧录导出
- `player.py` — mpv 播放器启动（带字幕）

### 字幕处理

`subtitle_processing.py` 负责：
- VTT/SRT 解析与写入
- ASS 字幕生成（支持中文和双语模式）
- 字幕分段合并（`split_source_cues`）
- 翻译后断句优化（`polish_translated_cues`，按 `target_line_width` 和 `max_subtitle_lines` 控制）

## 关键外部依赖

- `yt-dlp` — YouTube 视频/字幕下载
- `ffmpeg` — 音频抽取、字幕烧录、视频导出（位于 `.tools/ffmpeg-8.1/`）
- `mpv` — 本地视频播放器（位于 `.tools/mpv-20260307/`）
- `faster-whisper` — 本地语音转写（自动下载模型，首次使用较慢）
- `DeepSeek API` — 翻译引擎（需要 `DEEPSEEK_API_KEY`）

## 环境变量

参考 `.env.example`。核心变量：
- `DEEPSEEK_API_KEY` — DeepSeek 翻译 API 密钥（必须）
- `WHISPER_MODEL` / `WHISPER_FALLBACK_MODEL` — 转写模型选择
- `YTSUBVIEWER_DATA_ROOT` — 数据存储根目录（默认 `D:\YTSubViewerData`）
- `PREFER_AUTOMATIC_SUBTITLES` — 是否优先使用 YouTube 自动字幕

## 工作目录结构

每个翻译任务在 `workspace/jobs/{video_id}_{title_slug}/` 下生成：
- `job_state.json` — 完整任务状态（含源字幕、翻译字幕的 JSON 序列化）
- 视频文件、SRT/ASS 字幕文件、质量报告 Markdown

## 性能模式

三种模式（`fast` / `balanced` / `quality`）控制转写模型、翻译并行数和导出编码参数，在 `pipeline.py` 的 `_performance_profile()` 中定义。

## 图标规范

使用 Lucide Icons，禁止 Emoji。如需在前端新增图标元素，安装 `lucide` 并使用。

## 修改验证流程

修改后执行以下黑盒验证：
1. `python app.py` → 终端显示 uvicorn 启动日志，浏览器自动打开，无报错
2. 粘贴一个 YouTube 链接 → 元数据正确解析（标题、时长、字幕可用性）
3. 点击开始翻译 → 任务进入后台队列 → 进度实时更新 → 最终生成双语字幕文件
4. 导出视频 → ASS 字幕正确烧录，播放验证字幕与画面同步

---
> Source: [moneyperfect/YouTube-](https://github.com/moneyperfect/YouTube-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
