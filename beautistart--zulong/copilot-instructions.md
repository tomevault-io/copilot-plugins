## zulong

> > This file is auto-read by Qoder to provide project context.

# Zulong IDE - Project Intelligence

> This file is auto-read by Qoder to provide project context.

## Project Overview

Zulong IDE is a VS Code extension based on Cline v3.82.0, serving as the frontend for the Zulong multi-layer adaptive intelligent agent system. UI has been fully localized to Chinese (zh-CN).

## Dependencies

```bash
# Python 后端依赖 (使用 zulong_env/ 虚拟环境):
pip install -r requirements.txt
```

Key packages:
- **Kokoro-82M**: TTS 主引擎 (82M参数, CPU实时推理 <0.3s)
- **edge-tts**: TTS 备选引擎 (微软云端, 需网络)
- **CosyVoice3-0.5B**: TTS 备选, 需单独下载权重
- **SenseVoice-Small**: ASR 主引擎 (ONNX GPU, 含情感/事件检测, ~244M)
- **Whisper**: ASR 备选 (多语言 fallback, tiny/base/small)

## Build Pipeline

```bash
# Full build sequence (run from zulong-ide/ directory):
npm run protos                    # Generate TypeScript proto files
node esbuild.mjs --production     # Build extension with esbuild
npx @vscode/vsce package --no-dependencies --allow-missing-repository --skip-license  # Package VSIX
code --install-extension zulong-ide-0.1.0.vsix --force  # Install
```

## Key Architecture

- **Frontend**: Web 静态前端 (`zulong_web/static/`)
- **Extension**: TypeScript + esbuild (`zulong-ide/src/`)
- **Backend**: Python FastAPI + WebSocket (`zulong/ide/`)
- **Communication**: WebSocket at `ws://127.0.0.1:8090/ide`
- **Config**: `config/zulong_config.yaml` (port: 8090)

### 四层推理模型

```
L3 专家层 ──── DualBrainContainer(双脑热备) + 专家模型池
L2 认知层 ──── InferenceEngine(3934行) + FC循环 + TaskGraph(递归DAG) + CircuitBreaker(6信号)
L1 感知层 ──── L1-A反射/运动控制 + L1-B调度 + L1-C视觉 + L1-D听觉 + L1-E安全 → 输出(文本/语音/动作)
L0 设备层 ──── 摄像头/麦克风/扬声器/传感器驱动
```

### L1五个子层

| 子层 | 名称 | 核心模块 | 模型 | 意图识别 |
|------|------|---------|------|---------|
| L1-A | 感知与受控反射层 | ReflexController + AudioPreprocessor + FusionController + 运动控制(可接入厂家运动控制模块或端到端模型) | 可接入端到端运动模型 | 无(与L1-C/L1-D紧密协作) |
| L1-B | 调度与意图守门层 | Gatekeeper(2652行) + AttentionController + IntentFilter | ALBERT-tiny(15类) + VoiceIntentClassifier(3类) | 细粒度15类意图分类(与L1-C/D的交互意图判断不同) |
| L1-C | 静默视觉注意层 | OptimizedVisionProcessor + MobileNetV4-TSM + MediaPipeGestureRecognizer | YOLOv10+MediaPipe+MobileNetV4-TSM+可接入轻量视觉交互意图模型 | 交互意图判断(5类:WAVING/APPROACHING/GAZING/STILL/UNKNOWN) |
| L1-D | 听觉层 | L1D_VoicePlugin + L1D_AudioPlugin(三层注意力) | YAMNet+SenseVoice-Small(主ASR)+Whisper(备选) | 交互意图判断(基于事件标签的交互/非交互判断) |
| L1-E | 安全层 | L1E_GasPlugin | MQ-2烟雾传感器 | CRITICAL事件穿透 |

### L1-D听觉处理流水线
```
音频采集(L0) → 预加重+滤波(80Hz) → YAMNet(521类) → VAD → SenseVoice-Small单次推理(转录+情感+事件+语种) → 后处理意图判断 → L1-B串行协作(ALBERT 15类)
```
- 唤醒词: "你好"/"小紫"(NORMAL), "救命"(CRITICAL直达L1-B)
- SenseVoice-Small: ONNX INT8 + sherpa-onnx, 一次推理产出4类信息
- Whisper-tiny: 仅SenseVoice不可用时启用

### 三层注意力机制

**感知层三层注意力**(L1-C视觉和L1-D听觉均为此设计):
| 层级 | 职责 | 行为 |
|------|------|------|
| 无需注意 | 纯数据采集 | 无注意力，无事件产生 |
| 静默注意 | 持续推理 | 状态翻转时生成事件 |
| 交互注意 | 已生成事件 | 等待路由到L1-B |

**记忆动态注意力机制**(L2推理阶段):
- 全局注意(GLOBAL): 关注完整上下文
- 单链注意(SINGLE_CHAIN): 聚焦单条推理链
- 局部注意(FOCUS): 聚焦局部关键信息

### 意图识别体系
1. **L1-C交互意图判断**: MobileNetV4-TSM动作分类(可接入轻量视觉交互意图模型) → 5类(挥手/注视/靠近/静止/未知)
2. **L1-D交互意图判断**: SenseVoice事件标签后处理 → 交互/非交互二分
3. **L1-B细粒度意图分类**: ALBERT-tiny语义分类 → 15类(task_code/task_analysis/vision_query等)，与L1-C/D的交互意图判断不同
4. **L2**: 认知推理与决策，不做意图判断

### 记忆系统核心
- **MemoryGraph**(3547行): 异构图(NetworkX DiGraph), 11种节点+7种边, BFS扩散+赫布学习+艾宾浩斯衰减
- **双路径检索**: 热路径BFS(<50ms) + 冷路径FAISS(<200ms), asyncio.gather并行
- **三维标签**: Temperature(HOT/WARM/COLD) x Importance(6级) x TimeScope(RECENT/NON_RECENT)
- **经验库**: 混合检索(向量0.7+BM25 0.3) + 时间衰减 + 多标签过滤
- **FC工具暴露**: 4个工具(recall_memory/read_memory_node/save_memory_note/discover_related)暴露53个公开方法

### 层间通信
- **EventBus**: 单例, 优先级队列(CRITICAL>HIGH>NORMAL>LOW), 后台线程分发
- **SharedMemory**: L1插件持续写入多模态融合数据(SensorFusionData)
- **WebSocket**: IDE前端↔Python后端, 心跳30s, 超时5s, 3次丢失断连

## Critical File Map

| File | Role |
|------|------|
| `zulong-ide/src/core/api/providers/zulong.ts` | ZulongHandler - WebSocket API provider |
| `zulong-ide/src/core/api/transport/zulong-websocket.ts` | WebSocket transport layer |
| `zulong_web/static/index.html` | Zulong Web 主交互页面 |
| `zulong/launcher/web_chat_router.py` | Web 主通道与事件渲染数据源 |
| `zulong-ide/src/shared/providers/providers.json` | Provider list (Zulong is first entry) |
| `zulong/ide/ide_server.py` | Python backend entry (FastAPI + WebSocket) |
| `zulong/ide/ide_fc_runner.py` | FC loop executor (3556行) |
| `zulong/ide/ide_tool_registry.py` | Tool registry & smart routing |
| `zulong/l2/inference_engine.py` | Core inference engine (3934行, FC循环+记忆+5层防护) |
| `zulong/l2/circuit_breaker.py` | 6信号智能死循环检测器 (521行) |
| `zulong/l2/task_graph.py` | 递归DAG任务图 (1265行) |
| `zulong/memory/memory_graph.py` | 异构记忆图谱 (3547行, BFS+赫布+衰减+FAISS) |
| `zulong/l1b/scheduler_gatekeeper.py` | L1-B核心调度器 (2652行) |
| `zulong/l1b/intent_filter.py` | ALBERT意图过滤器 (15类) |
| `zulong/l1c/optimized_vision_processor.py` | L1-C四层级联视觉处理器 |
| `zulong/plugins/voice/l1d_audio_plugin.py` | L1-D三层注意力音频插件 |
| `zulong/plugins/voice/l1d_voice_plugin.py` | L1-D语音唤醒插件 |
| `zulong/plugins/gas/l1e_gas_plugin.py` | L1-E气体检测插件 |
| `zulong/models/audio_model_container.py` | 音频模型容器 (SenseVoice+Whisper) |
| `zulong/core/event_bus.py` | 事件总线 (单例, 优先级队列) |

## Model Ecosystem

| 模型 | 用途 | 规模 | 量化 | 推理引擎 |
|------|------|------|------|----------|
| YOLOv10-Nano | L1-C人体检测 | Nano | FP16 | PyTorch/CUDA |
| MediaPipe | L1-C姿态+手势(10种) | - | - | MediaPipe/CPU |
| MobileNetV4-TSM | L1-C动作分类(5类) | 轻量级 | - | PyTorch/CPU |
| YAMNet | L1-D环境音(521类) | 轻量级 | - | TF/CPU |
| SenseVoice-Small | L1-D ASR主引擎 | 244M | INT8 | ONNX GPU |
| Whisper-tiny | L1-D ASR备选 | 39M | - | PyTorch/CPU |
| ALBERT-tiny-chinese | L1-B意图(15类+3类) | 16M | - | PyTorch/CPU |
| Kokoro-82M | L3 TTS主引擎 | 82M | - | Kokoro/CPU |
| edge-tts | L3 TTS备选(云端) | 云端 | - | HTTP |
| CosyVoice3-0.5B | L3 TTS备选 | 0.5B | - | PyTorch |
| Qwen3.5-0.8B-int4 | L1-B/L2-BACKUP | 0.8B | INT4 | Transformers/GPU |
| Qwen3.5-2B-int4 | L2-CORE主推理 | 2B | INT4 | Transformers/GPU |
| bge-small-zh-v1.5 | Embedding向量 | 轻量级 | - | ST/CPU |
| DeepSeek-V4-Flash | L2云端推理 | 云端 | - | SiliconFlow API |

## Coding Conventions

- **Component function names must NOT be translated** (e.g., keep `RefreshButton`, not `刷新Button`)
- All user-facing strings are in Chinese (zh-CN)
- Removed features: Account page, Kanban modal (Cline native features not needed)
- Navigation tabs: Chat / MCP / History / Settings (4 tabs)

## Known Issues

- **Provider selection bug**: Selecting "Zulong (祖龙)" in settings dropdown may show Anthropic settings instead of Zulong WebSocket config. Investigate `handleProviderChange` in `ApiOptions.tsx`.

## TypeScript Check

```bash
cd zulong-ide && npx tsc --noEmit
```

## Documentation

- Usage guide: `docs/Zulong_IDE使用指南.md`
- Memory export: `docs/Qoder_Quest_Memory_Export.md`
- Deep analysis: `docs/祖龙系统深度技术分析报告.md`

## 测试记录 (2026-05-16)

### 通信通道测试结果

| 通道 | 端点 | 状态 | 说明 |
|------|------|------|------|
| Web端 | `ws://127.0.0.1:8090/ws` | ✅ 正常 | 简化协议，CHAT_MESSAGE/CHAT_RESPONSE |
| IDE端 | `ws://127.0.0.1:8090/ide` | ✅ 正常 | 完整FC协议，session_start/tool_result/task_complete |
| EventBus | 内部 | ✅ 正常 | 优先级队列，L1层内部+跨层通信 |

### IDE端消息协议

**前端→后端**：session_start / tool_result / user_cancel / ping / audio_chunk
**后端→前端**：session_ack / tool_request / display_text / display_reasoning / task_complete / task_error / status_update

### 已知问题

- **SiliconFlow API响应慢**：云端LLM可能限流，返回"响应较慢"fallback
- **解决方案**：启动Ollama本地模型 或 切换backend为transformer
- **Ollama模型**：qwen3.5:4b (L2_BACKUP), deepseek-v3.1:671b-cloud (L2_CORE)

### 插件安装

- VSIX位置：`zulong-ide/zulong-ide-0.1.0.vsix` (9.4MB)
- 安装命令：`code --install-extension zulong-ide-0.1.0.vsix --force`
- WebSocket配置：`ws://127.0.0.1:8090/ide`

---
> Source: [beautistart/zulong](https://github.com/beautistart/zulong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
