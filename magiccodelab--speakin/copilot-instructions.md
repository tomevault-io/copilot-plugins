## speakin

> SpeakIn 是一个静默运行的桌面语音输入工具。核心场景：用户按下全局热键 → 说话 → 松开或再按一次 → 语音被转为文字并自动输入到当前焦点窗口（IDE、浏览器、聊天框等）。全程无需切换窗口。

# SpeakIn 声入

## 产品定位

SpeakIn 是一个静默运行的桌面语音输入工具。核心场景：用户按下全局热键 → 说话 → 松开或再按一次 → 语音被转为文字并自动输入到当前焦点窗口（IDE、浏览器、聊天框等）。全程无需切换窗口。

**为什么做这个工具？**

面向超级个体、独立开发者、内容创作者。这些人每天需要向 AI 输出大量想法（需求描述、架构设计、Bug 分析等），动辄上千字。打字不仅慢，还对腱鞘和精力造成持续损伤。语音输入可以轻松达到每分钟 300 字以上的速度，并且更容易进入心流状态 —— 说话更像是在给 AI 下达指令，而不是在编辑文本。

**核心优势：**
- **速度**：语音输入 300+ 字/分钟，远超打字
- **零成本**：利用豆包、阿里云等厂商的免费 ASR 额度，日常使用基本不花钱
- **高准确率**：主流云端 ASR 的中文识别准确率已经非常高
- **AI 优化**：识别后的文字可经 AI 模型润色/翻译（如中文说话 → 英文输出），通过自定义提示词实现任意转换
- **静默工具**：后台运行，热键触发，不打断工作流

## 技术栈

- **后端**: Rust + Tauri 2
- **前端**: React 19 + TypeScript + Tailwind CSS 4 + Vite
- **ASR 供应商**: 豆包 (火山引擎二进制协议) / 百炼 (DashScope 双工流式) / 千问 (阿里云 Realtime API)
- **音频**: cpal (麦克风采集) + WASAPI Loopback (系统声音捕获) + rubato (重采样) + 自定义 RMS VAD
- **AI 优化**: 多供应商支持 (OpenAI 兼容 API)，自定义提示词，流式输出
- **输入模拟**: enigo (键盘模拟) + arboard (剪贴板)
- **热键**: windows-sys (原生 WH_KEYBOARD_LL 低级键盘钩子)
- **配置**: tauri-plugin-store (JSON) + OS Keyring (凭据)
- **动画**: motion (React 动画库)

## 项目结构

```
src/                              # React 前端
  App.tsx                         # 主组件，状态管理，事件监听，overlay-phase 广播
  components/
    VoicePanel.tsx                # 录音按钮 + 音频可视化波形
    Settings.tsx                  # 设置面板 (多 Tab)
    RecordingOverlay.tsx          # 系统级录音指示覆盖层 (独立窗口，可拖动)
    OnboardingDialog.tsx          # 首次启动引导弹窗
    NetworkLog.tsx                # 网络日志面板
    AboutDialog.tsx               # 关于对话框
    CloseDialog.tsx               # 关闭行为对话框
    settings/
      AsrSettingsTab.tsx          # ASR 供应商 + 音频来源设置
      AiOptimizeTab.tsx           # AI 优化设置
      AiProvidersTab.tsx          # AI 供应商管理
      PromptsTab.tsx              # 提示词管理
      StatsTab.tsx                # 使用统计
  lib/
    overlay.ts                    # 覆盖层窗口管理 (位置持久化)
    sounds.ts                     # 音效播放
    hotkey.ts                     # 热键字符串解析
    ai-providers.ts               # AI 供应商配置类型
    prompts.ts                    # 提示词模板类型
    theme-colors.ts               # 主题色管理
    replacements.ts               # 文本替换规则

src-tauri/src/                    # Rust 后端
  lib.rs                          # Tauri 命令，应用状态，录音生命周期
  asr/                            # ASR 供应商抽象层
    mod.rs                        # AsrProvider trait + wait_for_speech (30s 超时)
    doubao.rs                     # 豆包 ASR (火山引擎二进制协议)
    dashscope.rs                  # 百炼 ASR (DashScope 双工流式)
    qwen.rs                       # 千问 ASR (阿里云 Realtime API)
    protocol.rs                   # 火山引擎二进制协议编解码
  ai/                             # AI 优化模块
    mod.rs                        # AI 优化入口 + 供应商管理
    client.rs                     # OpenAI 兼容 API 客户端
  audio.rs                        # 麦克风管理，VAD，重采样，AudioFrame
  loopback.rs                     # WASAPI Loopback 系统声音捕获
  hotkey.rs                       # Windows 全局热键 (低级键盘钩子)
  input.rs                        # 文本输出 (粘贴/打字模拟)
  storage.rs                      # 设置存储 (JSON store + OS Keyring)
  filler.rs                       # 语气词过滤
  replacements.rs                 # 文本替换规则
```

## 核心架构

### 音频管道

```
音频来源 (二选一):
  MicrophoneManager (cpal)     ──→  VAD 过滤 → 重采样 16kHz → AudioFrame
  LoopbackCapture (WASAPI)     ──→  VAD 过滤 → 重采样 16kHz → AudioFrame
                                           ↓
                                      audio_tx 通道
                                           ↓
                              wait_for_speech (30s 无语音超时)
                                           ↓
                                   ASR Provider (WebSocket)
                                           ↓
                              transcription-update 事件 → 前端
                                           ↓
                              AI 优化 (可选) → send_text_input → 焦点窗口
```

### 录音会话生命周期
1. 热键/按钮触发 → `do_start_recording_impl()`
2. 根据 `audio_source` 选择麦克风或系统声音捕获
3. 创建 `(stop_tx, stop_rx)` + `(audio_tx, audio_rx)` 通道
4. `wait_for_speech()` 等待首个语音帧（30 秒超时，不连接 ASR 不计费）
5. 检测到语音 → 连接 ASR WebSocket → 音频流式发送 → 实时返回转写
6. 停止（手动/自动 6 秒静音）→ 发送结束包 → 等待 FINAL → 输出文字

### VAD 策略
- **阈值**: RMS 150 (~-46 dBFS)，非常低，只过滤纯环境噪音
- **trailing**: 语音结束后继续发送 200ms，避免尾部截断
- **预热**: 前 1 秒无条件发送，让 ASR 准备好
- **自动停止**: 检测到语音后连续 6 秒静音 → 自动结束会话（仅麦克风模式）
- **无语音超时**: 30 秒无语音 → 自动取消（不连接 ASR，零计费）

### Overlay 状态机
通过 `overlay-phase` 事件（带 sessionId）统一管理，App.tsx 为唯一真值源：
- `recording` → 正在录音
- `processing` → ASR 处理中
- `optimizing` → AI 优化中
- `idle` → 完成/关闭

### 前端事件流
- `recording-status` (bool): 录音状态变化
- `overlay-phase` ({ phase, sessionId }): 覆盖层状态
- `transcription-update` (JSON): 转写结果 (`is_final` 区分确认/临时文本)
- `asr-error` (string): ASR 错误
- `connection-status` (bool): WebSocket 连接状态
- `audio-level` (f32): 音频电平 (UI 波形)
- `ai-optimize-chunk` ({ chunk, session_id }): AI 优化流式输出

## 开发规范

- 所有用户可见文本使用中文
- 通用配置存储在 tauri-plugin-store (JSON)，敏感凭据存储在 OS Keyring
- Rust 代码优先考虑性能，避免不必要的分配
- 前端使用 CSS 变量主题系统 (`--t-fast`, `--t-base` 等过渡时间)
- 录音停止时的关键顺序：stop_tx → loopback.stop() / mic.stop_forwarding() → 状态更新
- Overlay 窗口状态由 `overlay-phase` 事件驱动，RecordingOverlay 不独立管理状态
- loopback.stop() 必须在释放 AppState 锁之后调用（避免 thread::join 阻塞）

## 构建

```bash
pnpm install
pnpm tauri dev      # 开发模式
pnpm tauri build    # 生产构建
```

## 常见问题

- **45000081 超时**: 确保结束包正确发送，检查 ASR 循环退出路径
- **麦克风权限**: Windows 设置 > 隐私和安全 > 麦克风
- **VAD 阈值**: `audio.rs` 中 `threshold: 150.0` (RMS), 静音自动停止 6 秒
- **Overlay 卡住**: 检查 `overlay-phase` 事件是否在所有路径上正确发射
- **系统声音无输出**: 确认默认输出设备有音频播放，WASAPI Loopback 需要活跃的音频流

---
> Source: [magiccodelab/speakin](https://github.com/magiccodelab/speakin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
