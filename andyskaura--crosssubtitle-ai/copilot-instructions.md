## crosssubtitle-ai

> CrossSubtitle-AI 技术需求与开发规划书

CrossSubtitle-AI 技术需求与开发规划书
1. 项目愿景
CrossSubtitle-AI 的目标是打造一款 极简、高性能、完全本地优先 的视频转录与翻译工具，面向多语言视频内容生产场景，最终输出高质量字幕文件。

核心价值：

本地运行，保护音视频内容隐私
利用 AI PC 的 GPU / NPU / CoreML 加速，实现高吞吐推理
减少 Whisper 在静音、背景音、纯音乐场景下的“幻听”问题
支持全球主流语言识别，并稳定转换为中文或英文字幕
提供实时预览、可编辑、可导出的完整字幕工作流
2. 技术目标
2.1 功能目标
支持视频/音频文件导入
自动抽取音频并标准化预处理
基于 VAD 切分有效语音区间
执行本地 Whisper 转录/翻译
根据目标语言选择不同翻译策略
实时展示字幕生成进度
支持导出 SRT / ASS / VTT
2.2 非功能目标
启动快、包体小、资源占用可控
支持 macOS / Windows
推理链路可扩展，便于后续接入更多模型
模块解耦，便于替换 VAD、ASR、翻译后端
前后端通信稳定，可观察任务状态和错误信息
3. 技术栈选型
模块	技术选型	说明
表现层（UI）	Vue 3 + Vite + Tailwind CSS + Pinia	轻量、开发效率高、组件化好
容器层（Desktop）	Tauri v2 + Rust	包体小、性能高、系统能力强
音频处理	FFmpeg Sidecar	统一抽流与重采样
语音活动检测	Silero VAD + ONNX Runtime (ort)	过滤静音/噪声/纯音乐，提高识别质量
转录引擎	whisper-rs	本地推理，支持多平台硬件加速
翻译引擎	Whisper Native / OpenAI-compatible API / 本地 LLM	按目标语言动态选择策略
字幕导出	Rust 原生实现	统一输出 SRT / ASS / VTT
4. 系统架构设计
整体处理链路如下：

用户导入视频或音频文件
Rust 调用 FFmpeg 提取并转码为标准 PCM
VAD 模块分析音频，生成语音片段时间轴
Whisper 模块对片段执行转录或直译
若目标为中文，则进入 LLM 翻译链路
实时将结果推送给 Vue 前端展示
用户校对字幕并导出目标格式
5. 核心模块设计
5.1 音频预处理管线
输入
视频文件：mp4 / mkv / mov / avi
音频文件：mp3 / wav / m4a / flac
处理步骤
1. FFmpeg 抽流与标准化
统一转换为：

采样率：16kHz
位深：16bit
声道：mono
输出格式：PCM 或 WAV
建议命令逻辑：

ffmpeg -i input.mp4 -ac 1 -ar 16000 -f wav output.wav
2. PCM 解码与缓存
Rust 读取 WAV/PCM 数据，转换为：

Vec<f32>
采样值范围归一化至 [-1.0, 1.0]。

3. Silero VAD 检测
使用 ort 加载 silero_vad.onnx，输出语音片段：

Vec<(f32, f32)>
每个元组表示：

起始时间（秒）
结束时间（秒）
预期收益
跳过静音与非语音段
降低 Whisper 幻听概率
减少无效推理时长
提升整体处理速度
5.2 智能转录与翻译引擎
路由策略
路径 A：源语言 -> 英文
直接使用 Whisper 原生翻译能力：

task = translate
适用场景：

非英语语音转英文字幕
对实时性要求较高
尽量减少外部翻译调用
路径 B：源语言 -> 中文
分两步：

Whisper 转录原文
task = transcribe
将原文批量送入 LLM 翻译为中文
适用场景：

非英语内容转中文
需要更自然的语义表达
对专有名词、人称、上下文一致性要求更高
5.3 上下文关联翻译
为避免逐句翻译导致语义断裂，翻译模块采用 滑动窗口策略。

策略设计
每批发送 10~15 条字幕
携带上一批末尾若干条作为上下文
保留角色称呼、专有名词、代词指代一致性
示例结构
{
  "context": [
    "上一段末尾字幕1",
    "上一段末尾字幕2"
  ],
  "segments": [
    "当前待翻译字幕1",
    "当前待翻译字幕2"
  ]
}
目标
提升“你/您”“他/她”等代词一致性
减少断句错误
提高术语统一性
5.4 实时同步与前端预览
Rust 后端在每个阶段通过事件通知前端：

文件入队
抽流进度
VAD 进度
Whisper 推理进度
翻译进度
单条字幕产出
任务完成/失败
Tauri 事件建议：

window.emit("task:progress", payload)
window.emit("task:segment", payload)
window.emit("task:error", payload)
window.emit("task:done", payload)
前端 UI 应支持：

多文件任务队列
当前任务状态可视化
实时字幕流式展示
双栏对照编辑
时间戳定位
5.5 字幕导出模块
支持格式
SRT
ASS
VTT
输出要求
时间轴准确
支持多语言文本
保留编辑后的最终内容
ASS 可扩展样式配置
字幕数据结构建议统一为：

type SubtitleSegment = {
  id: string
  start: number
  end: number
  sourceText: string
  translatedText?: string
}
6. 数据流设计
建议建立统一任务模型：

type TaskStatus =
  | 'queued'
  | 'extracting'
  | 'vad_processing'
  | 'transcribing'
  | 'translating'
  | 'completed'
  | 'failed'
type SubtitleTask = {
  id: string
  filePath: string
  fileName: string
  sourceLang?: string
  targetLang: 'zh' | 'en'
  status: TaskStatus
  progress: number
  segments: SubtitleSegment[]
  error?: string
}
7. 模块划分建议
Rust 侧
audio.rs
负责：

调用 FFmpeg
管理音频抽取与格式标准化
输出 WAV/PCM 文件路径
vad.rs
负责：

加载 ONNX Runtime
执行 Silero VAD
返回语音时间片段
whisper.rs
负责：

加载 Whisper 模型
执行转录/翻译
汇报处理进度
translate.rs
负责：

对接 OpenAI-compatible API 或本地模型
管理上下文窗口翻译
返回中文结果
subtitle.rs
负责：

统一字幕结构
导出 SRT / ASS / VTT
task.rs
负责：

管理任务生命周期
协调各模块串联
向前端广播事件
Vue 侧
页面建议
文件拖拽上传页
任务队列面板
字幕预览与编辑面板
导出设置面板
Pinia Store 划分
useTaskStore
useSubtitleStore
useSettingsStore
8. 开发路线图
第一阶段：环境与音频引擎（Week 1）
Rust
集成 tauri-plugin-shell
打通 FFmpeg sidecar 调用
实现 audio.rs
实现 vad.rs 基础版本
Vue
搭建基础界面
实现多文件拖拽上传
建立任务队列状态管理
交付物
可导入文件
可抽取音频
可生成 VAD 时间片段
第二阶段：Whisper 推理核心（Week 1-2）
Rust
实现 whisper.rs
支持模型动态加载
支持 large-v3-turbo 及量化模型
加入平台加速检测逻辑
Vue
显示 VAD / 转录进度条
展示实时字幕输出
交付物
完整本地转录链路可跑通
前端实时看到字幕增量结果
第三阶段：翻译中枢（Week 2）
Rust
实现 translate.rs
对接 OpenAI-compatible 接口
实现滑动窗口翻译缓存
Vue
双栏字幕编辑器
支持原文/译文联动显示
点击时间戳跳转定位
交付物
中英目标语言翻译流程打通
具备可编辑字幕界面
第四阶段：优化与打包（Week 3）
性能
优化批处理与内存占用
Windows 侧尝试 Vulkan 加速
macOS 侧适配 CoreML
工程化
增加错误处理与日志
配置 Github Actions
自动构建 Windows / macOS 安装包
交付物
可分发桌面应用
基本可用的发布版本
9. 风险与关键问题
1. Whisper 幻听问题不能只靠模型解决
必须结合：

VAD 过滤
合理切片
静音抑制
长音频分段策略
2. 不同平台硬件加速差异较大
需要预留能力检测和回退机制：

可用 GPU/NPU 就启用
不可用时回退 CPU
前端明确提示当前推理模式
3. 翻译质量依赖上下文设计
中文输出质量主要取决于：

分批粒度
上下文拼接方式
专有名词缓存策略
4. 超长文件需要任务恢复与断点信息
后续可考虑：

中间结果缓存
崩溃恢复
已完成片段复用
10. 第一版 MVP 范围建议
为了尽快落地，建议第一版先做：

单文件处理
本地 Whisper 转录
VAD 过滤
英文直译 / 中文二段翻译
SRT 导出
简单实时预览
先不做：

视频预览播放器
ASS 高级样式编辑
批量复杂任务编排
云端模型管理
自动术语表
11. 给 Codex 的可执行 Prompt
模块 A：VAD 实现
请在 Rust 的 Tauri v2 项目中实现一个 `src-tauri/src/vad.rs` 模块，使用 `ort` 加载 Silero VAD ONNX 模型。输入为 `Vec<f32>` 格式的 16kHz 单声道音频采样，输出为 `Vec<(f32, f32)>`，表示检测到语音的开始和结束秒数。请封装成结构清晰、内存安全的 API，并处理模型初始化、推理失败和阈值配置。
模块 B：Whisper 命令封装
请基于 `whisper-rs` 为 Tauri v2 编写一个 Rust Command。命令接收 WAV 文件路径、源语言、目标语言和任务 ID。如果目标语言是英文，则使用 Whisper 的 `translate` 任务；否则使用 `transcribe`。处理过程中通过 `window.emit` 持续向前端发送进度事件和分段结果事件。请把实现拆分到 `src-tauri/src/whisper.rs`，并提供可复用的数据结构。
模块 C：翻译滑动窗口
请在 Rust 中实现 `src-tauri/src/translate.rs`，对接 OpenAI-compatible Chat Completions 接口。输入是一组带时间轴的字幕片段，目标是将原文翻译成中文。要求实现滑动窗口翻译：每批 10 到 15 条，并附带上一批末尾若干条作为上下文，以保证人称和术语一致。请返回结构化结果，并处理接口超时、重试和 JSON 解析失败。
模块 D：字幕导出
请在 Rust 中实现一个 `subtitle.rs` 模块，接收统一的字幕片段结构，支持导出为 SRT、VTT 和 ASS 三种格式。要求时间格式正确，支持多语言文本，并为 ASS 预留基础样式配置。

---
> Source: [AndySkaura/crosssubtitle-ai](https://github.com/AndySkaura/crosssubtitle-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
