## mlx-live

> 本文件为进入本仓库协作的智能体提供中文初始化说明。

# AGENTS.md

本文件为进入本仓库协作的智能体提供中文初始化说明。

## 项目概览

这是一个基于 MLX 的本地实时语音助手示例，后端使用 FastAPI + WebSocket，前端是一个纯静态网页。

当前能力包括：

- 浏览器端麦克风采集
- 16k PCM 音频上传
- Silero VAD 语音活动检测
- MLX ASR 语音识别
- MLX VLM 文本生成
- MLX TTS 流式语音播报
- 插话打断：用户重新说话时，后端取消当前回复，前端立即停播

## 主要文件

- `main.py`
  后端主入口。负责模型加载、WebSocket 会话、VAD、ASR、VLM、TTS 和打断逻辑。
- `web/index.html`
  前端页面，负责界面、WebSocket 通信、录音和音频播放。
- `web/pcm-recorder-worklet.js`
  浏览器端 `AudioWorklet`，负责把麦克风输入重采样并输出 PCM16 数据。
- `custom_voice.wav`
  当前 TTS 参考音色文件。
- `pyproject.toml`
  Python 依赖声明。
- `uv.lock`
  `uv` 锁文件。
- `README.md`
  面向用户的中文说明文档。

## 运行方式

安装依赖：

```bash
uv sync
```

启动服务：

```bash
uv run python main.py
```

默认访问地址：

```text
http://127.0.0.1:8000
```

## 开发约定

- 默认使用中文沟通和写说明，除非用户明确要求英文。
- 修改后端时，优先检查：
  - VAD 输入帧长是否满足 Silero 要求
  - WebSocket 事件类型是否与前端保持一致
  - 打断逻辑是否会让旧的音频或旧的回复继续泄漏到前端
- 修改前端时，优先检查：
  - `AudioWorklet` 是否仍输出 16k PCM
  - 播放队列在 `interrupt` 事件后是否会被清空
  - 浏览器缓存是否可能导致旧脚本仍在运行
- 如果调整依赖，记得同步更新：
  - `pyproject.toml`
  - `uv.lock`

## 验证建议

常用静态检查：

```bash
python3 -m py_compile main.py
node --check web/pcm-recorder-worklet.js
```

如果修改了 `web/index.html` 里的内联脚本，可以把 `<script>` 内容抽出来再做一次 `node --check`。

## 已知风险

- 项目主要面向本地 Apple Silicon + MLX 环境，不是通用部署模板。
- 浏览器端的回声消除和设备环境会直接影响 VAD 打断效果。
- 更新 `AudioWorklet` 后，浏览器可能缓存旧版本，联调时需要强制刷新页面。
- 目前没有完整自动化测试，主要依赖手工联调。

## 协作建议

- 优先做最小改动，避免破坏当前可用的语音链路。
- 如果修改打断逻辑，要同时检查：
  - 后端是否取消了旧任务
  - 前端是否停掉了已排队播放的音频源
- 不要随意替换 `custom_voice.wav`，除非任务明确要求变更音色。

---
> Source: [akazwz/mlx-live](https://github.com/akazwz/mlx-live) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
