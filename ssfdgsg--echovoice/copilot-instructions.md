## echovoice

> > 目标：做一个“像 Soundpad 那样好用”的音效板，但我们采用**用户态**方案：

# EchoVoice — Soundboard + Mic Mixer (VB-Cable) 桌面应用项目说明

> 目标：做一个“像 Soundpad 那样好用”的音效板，但我们采用**用户态**方案：  
> **捕获真实麦克风 → 混入音效 → 输出到 VB-Audio Virtual Cable（VB-Cable）→ 目标应用把它当麦克风收音**。  
> GUI 追求美观、现代、低占用，并能打包成 Windows `.exe` 安装包（开包即用）。

---

## 1. 项目代号与一句话描述

**项目代号**：`EchoVoice`  
**一句话描述**：在 Windows 上把“真实麦克风 + 音效”实时混音后，输出到 `VB-Cable` 虚拟麦克风，让游戏/语音软件像用正常麦克风一样接收。

---

## 2. 目标与成功标准

### 2.1 核心目标（MVP 必须达成）
1. **低学习成本**：用户只需要完成一次“驱动安装 + 设备选择”（或一键向导），即可长期使用。
2. **真实麦克风可用**：即便不播放音效，也能把真实麦克风完整转发给目标应用（相当于“桥接”）。
3. **音效混入**：点击按钮 / 全局快捷键触发音效，音效被叠加进麦克风流输出。
4. **低延迟**：语音聊天可接受（尽量做到 < 50ms 级别；具体依赖设备 buffer/系统负载）。
5. **资源占用可控**：常驻托盘运行；前端轻量；后端实时线程不做频繁分配。
6. **可发布**：可打包成 Windows 安装程序（`-setup.exe` 或 `.msi`），用户安装后能运行。

### 2.2 体验目标（强烈建议）
- **托盘常驻**：开机自启（可选）、右键菜单、快速开关。
- **快捷键体系**：全局快捷键触发音效、停止、静音、Push-to-Talk/Push-to-Play。
- **设备与状态可视化**：VB-Cable 是否安装、当前输出设备、采样率、缓冲健康度、丢包统计。

---

## 3. 非目标（明确不做/暂缓）
- ❌ 自研“虚拟麦克风驱动”（内核级驱动开发成本过高，不在本项目范围）。
- ❌ 试图“hook 系统麦克风驱动数据流”做无缝注入（同上）。
- ❌ 追求多平台完整等价（macOS/Linux 未来可评估，但 MVP 只做 Windows + VB-Cable）。

---

## 4. 核心约束与“免配置”现实边界

### 4.1 为什么一定需要 VB-Cable 或类似虚拟设备
要让**任意游戏/语音软件**都能收到“我们生成的麦克风音频流”，必须有一个系统可见的“录音设备/麦克风端点”。  
VB-Cable 提供了这种虚拟 I/O 通道（`CABLE Input` / `CABLE Output`）。

### 4.2 “不配置”的可实现程度
在**不写驱动**的前提下，完全 0 配置基本做不到，但可以做到“接近 0 配置”：

- **最低配置（现实可落地）**：
  1) 安装 VB-Cable（驱动）  
  2) 把目标应用的麦克风选择为 `CABLE Output`（一次性）  
  3) 我们的软件常驻托盘，持续桥接真实麦克风 → VB-Cable，并在需要时混入音效

- **进一步降低配置成本（推荐实现）**：
  - 首次启动向导：检测 VB-Cable 是否存在，不存在则引导安装；存在则指导用户把“默认麦克风/通信设备”设为 `CABLE Output`。  
  - （可选）提供“一键打开 Windows 声音设置”的按钮（不用我们做系统级修改也能很快完成）。

---

## 5. 技术选型（最终推荐）

### 5.1 GUI：Tauri v2（首选）
**理由**：相比 Electron，Tauri 不捆绑浏览器运行时，使用系统 WebView（Windows 上为 WebView2），通常更轻量、占用更低；同时 UI 仍可用现代 Web 技术实现“很好看”的界面。

**前端推荐栈**（任选其一）：
- `SvelteKit + TailwindCSS + shadcn-svelte`（开发快、UI 现代、体积小）
- 或 `React + Tailwind + Radix UI`
- 暗色模式优先

### 5.2 音频后端：Rust（WASAPI）
**实现策略**：用户态 WASAPI
- Capture：从“真实麦克风设备”以共享模式采集 PCM
- Render：把混音后的 PCM 以共享模式写入 `VB-Cable` 的渲染端点（`CABLE Input`）

**Rust 库建议**：
- `wasapi` crate：对 WASAPI 的相对安全封装（接近原生 API 的结构）  
- 音效解码：`symphonia`（支持多格式）  
- 重采样：`rubato`（实时处理友好，可预分配缓冲）
- 锁自由/低锁 buffer：`ringbuf` 或自研固定容量环形缓冲
- 配置：`serde` + `serde_json`
- 日志：`tracing` + `tracing-subscriber`

### 5.3 全局快捷键：Tauri global-shortcut 插件（必选）
用于注册系统级快捷键触发音效、停止、切换静音等。

### 5.4 系统托盘：Tauri 内置 Tray（建议）
常驻托盘、右键菜单、显示当前状态。

---

## 6. 总体架构

```text
┌──────────────────────────────────────────────────────────┐
│                      Tauri 前端 UI                        │
│  Sound 列表/快捷键/音量/设备/状态/托盘菜单/向导            │
└───────────────┬──────────────────────────────────────────┘
│ IPC（tauri::command / event）
┌───────────────▼──────────────────────────────────────────┐
│                     Rust 后端（Audio Engine）             │
│  - Device manager：枚举、选择、热插拔重连                 │
│  - Mic Capture：WASAPI capture（共享模式）                │
│  - FX Player：音效解码/预加载/触发                         │
│  - Mixer：mic + fx 混音、增益、限幅、统计                  │
│  - VB Render：WASAPI render 写入 CABLE Input              │
│  - Telemetry：buffer health、XRuns、延迟估计、日志         │
└──────────────────────────────────────────────────────────┘
```

---

## 7. 音频引擎设计细节（可直接照此实现）

### 7.1 设备选择策略
- **捕获设备（Mic）**：默认使用系统默认输入设备；允许 UI 手动选择具体设备 ID。
- **输出设备（VB Render）**：优先自动匹配名称包含 `CABLE Input` 的设备；允许 UI 手动选择。
- **目标应用使用的麦克风**：`CABLE Output`（这是目标应用侧看到的“麦克风”）。

> 备注：设备名称可能因系统语言略有差异，但通常会包含 “CABLE Input / CABLE Output”。

### 7.2 WASAPI 模式
- **共享模式（Shared）优先**：不抢占设备，减少与其他应用冲突概率。
- 建议使用**事件驱动**（Event-driven buffering）降低延迟与 CPU 占用（实现复杂度略高，但值得）。

### 7.3 音频格式（Format）与转换
- 以 **VB 渲染端点的 mix format** 作为“全局内部输出格式”（例如 48kHz、2ch、f32 或 i16）。
- 捕获到的麦克风格式可能不同：需要转换到输出格式：
  - 采样率不同 → `rubato` 重采样
  - 通道数不同（mono→stereo）→ 复制到两个通道或按声像规则
  - sample format 不同（i16↔f32）→ 转换

**建议内部统一为 `f32` 混音**，最终写入渲染缓冲时再转回目标格式（如果目标不是 f32）。

### 7.4 缓冲与线程模型（强推荐）
- **Capture 线程**：从 `IAudioCaptureClient` 拉取 packet → 转换格式 → 写入 `mic_ring_buffer`
- **Render 线程**：根据 `IAudioClient::GetCurrentPadding` 计算可写帧数 → 从 `mic_ring_buffer` 读帧 → 混入 `fx_active_list` → 写入 `IAudioRenderClient`

关键点：Render 线程必须**避免动态分配**和长时间锁；所有 buffer 尽量预分配。

### 7.5 混音算法（最小可用版）
- 每帧每通道：
  - `out = mic_gain * mic + fx_gain * sum(active_fx_samples)`
  - 简单限幅：`out = out.clamp(-1.0, 1.0)`
- 进阶可选：
  - soft-clip：例如 `tanh` 或自定义 soft limiter（减少爆音感）
  - ducking：播放音效时自动把 mic 降低一点（“旁白压音乐”）

### 7.6 音效播放（Soundboard）
- 音效文件导入后：
  - 用 `symphonia` 解码到 PCM
  - 转换为“内部输出格式”（采样率/通道/样本格式）
  - 缓存在本地（内存或磁盘缓存 + 内存映射视情况）
- 触发音效：
  - 创建一个 `FxInstance { sound_id, pos, gain, loop?, stop_on_release? }`
  - Render 线程每次填充 buffer 时推进 pos
- 支持并发多音效（多个 FxInstance 同时叠加）

### 7.7 “只在用软件时才输出音频”的实现方式（推荐做法）
我们采用“桥接常开”的方式最稳定：
- 软件常驻托盘时：持续把真实麦克风转发到 VB-Cable（所以目标应用一直能正常收音）
- 当用户触发音效时：在同一路输出中混入音效即可

> 这比“临时切换设备”更稳定：因为很多游戏/语音软件不会在运行中频繁重新打开输入设备。

---

## 8. GUI/交互设计（美观 + 易用）

### 8.1 主窗口信息架构
- 左侧：音效分类/搜索
- 中间：音效列表（名称、时长、绑定快捷键、收藏）
- 右侧：当前输出状态
  - Mic 输入设备
  - VB 输出设备
  - Mic 音量滑块
  - FX 音量滑块
  - “测试播放（本地）”
  - “输出已连接/未连接”状态灯
  - buffer 健康度（绿色/黄色/红色）

### 8.2 系统托盘（必做）
- 左键：显示/隐藏主窗口
- 右键菜单：
  - 开关桥接（Bridge On/Off）
  - 一键静音（Mic Mute / FX Mute）
  - 快速播放最近音效
  - 打开设置 / 退出

### 8.3 全局快捷键（必做）
- Play sound #1/#2/...
- Stop all sounds
- Toggle mute
- Push-to-Play（按住才混入音效/或按住才输出）

---

## 9. 配置与数据存储

### 9.1 配置文件（JSON）
保存到系统应用数据目录，例如：
- `config.json`
- `sounds/`（用户导入的音效资源，可复制到应用目录或保留原路径）

建议 schema（示例）：
```json
{
  "devices": {
    "mic_device_id": "default",
    "vb_render_device_id": "auto"
  },
  "levels": {
    "mic_gain": 1.0,
    "fx_gain": 0.8
  },
  "hotkeys": [
    { "sound_id": "airhorn", "accelerator": "Ctrl+Shift+1" }
  ],
  "ui": {
    "theme": "dark",
    "start_minimized": true
  }
}
```

---

## 10. 实现路线（按里程碑推进）

### M0：工程脚手架

* Tauri v2 项目初始化
* 前端 UI 框架（SvelteKit/React）+ 基础布局
* 后端 Rust commands + 状态 event 通道
* 系统托盘最小版本
* 全局快捷键插件接入（注册/注销最小示例）

### M1：音频“桥接”跑通（最关键）

- [x] 枚举音频设备
- [x] 选择真实麦克风作为 capture
- [x] 自动找到 `CABLE Input` 作为 render
- [x] 实现 capture → format convert → ring buffer → render
- [ ] UI 显示：已连接/错误原因
- [ ] 统计：buffer underrun 次数

验收标准：

* 把 Discord/游戏麦克风设为 `CABLE Output` 后，能听到真实麦克风语音。

### M2：音效混入（Soundboard MVP）

- [x] 导入 WAV（先只支持 WAV，稳定后再扩格式）
- [x] 音效缓存为内部格式
- [x] UI 一键播放
- [x] 混音叠加成功（对端能听到）

### M3：快捷键绑定 + 托盘工作流

* 音效绑定全局快捷键
* 托盘快速操作（Mute/Stop/Toggle）
* 最小“首次启动向导”：检测 VB-Cable、提示把输入设为 `CABLE Output`

### M4：完善与发布

* 支持 MP3/OGG 等（symphonia）
* 重采样与通道转换健壮性（rubato）
* 打包 Windows 安装程序（NSIS setup.exe / WiX msi）
* 崩溃恢复、设备断开重连
* 性能优化（减少分配、锁、日志限流）

---

## 11. 构建、运行与打包

### 11.1 开发运行

* 前端依赖安装：`pnpm install`（或 npm/yarn）
* 开发启动：`pnpm tauri dev`

### 11.2 打包发布

* `pnpm tauri build`
* Windows 目标：

  * `NSIS` 生成 `*-setup.exe`
  * 或 `WiX` 生成 `.msi`

> WebView2：安装包需确保用户系统具备 WebView2 Runtime；可选择 “下载引导/嵌入引导/离线安装器” 等模式（体积与是否需要联网取舍）。

---

## 12. 测试计划（务实可执行）

### 12.1 单元测试（Rust）

* 混音：叠加、增益、限幅
* 格式转换：i16↔f32，mono↔stereo
* 重采样：输入输出帧数与边界条件
* ring buffer：读写正确性

### 12.2 手工回归（Windows）

* 设备热插拔：拔掉麦克风、切换默认设备、VB-Cable 重装
* 多音效并发：同时触发多个音效不崩溃
* 长时间运行：托盘常驻 2h 不漂移、不爆内存

### 12.3 端到端验收脚本（建议写成 checklist）

* Discord：输入设为 `CABLE Output`，测试语音 + 音效
* 游戏内语音：同上
* 录音软件：选择 `CABLE Output` 录到混音后的效果

---

## 13. 风险与应对

1. **目标应用不重新打开麦克风设备**

   * 应对：推荐“桥接常开 + 把默认麦克风设为 CABLE Output”的稳定策略
2. **延迟/爆音**

   * 应对：事件驱动 WASAPI、预分配、软限幅、降低 buffer underrun
3. **VB-Cable 不存在或设备名不同**

   * 应对：提供设备枚举选择 + 自动匹配失败时提示
4. **分发与许可风险（VB-Cable）**

   * 应对：默认不把 VB-Cable 驱动打包进安装器；仅检测并引导用户从官方渠道安装；如需商业/组织分发，先确认 VB-Audio 授权条款

---

## 14. 代码规范（Claude/协作者必须遵守）

* 音频实时线程（Render/Capture）：

  * 禁止频繁分配（Vec push/resize）/ 禁止长时间锁
  * 日志限流（不要每帧 log）
* UI 与后端通信：

  * command 只做控制/查询；大流量音频数据不走 IPC
* 每次改动保持大体整齐：

  * Rust：`cargo fmt` / `cargo clippy`（可选）
  * 前端：eslint/prettier（如启用）
* 对外行为要可解释：

  * UI 中明确提示“目标应用需使用 CABLE Output 作为麦克风”
* 不要把任何第三方驱动/安装包直接加入仓库（除非许可明确允许）

---

## 15. Claude（或任何 AI 编码助手）工作指令

当你被要求实现/修改功能时，请按以下顺序工作：

1. 先确认当前里程碑（M0~M4）属于哪一阶段，避免超前堆功能
2. 优先让“音频桥接链路稳定运行”（M1 是地基）
3. 所有音频线程相关的改动必须注明：

   * 是否新增分配、是否新增锁、是否影响实时性
4. 提交（或输出补丁）前，至少确保：

   * 项目能编译（前端 build + Rust 编译）
   * 关键路径不 panic
5. 任何“需要用户额外配置”的点都要在 UI/文档里显式写出来

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ssfdgsg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ssfdgsg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
