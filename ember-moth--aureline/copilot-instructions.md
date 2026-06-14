## aureline

> 本文面向 Codex、Claude、Gemini、Qwen 等自动化开发 agent。假设你没有任何历史上下文，

# Aureline Agent 接手指南

本文面向 Codex、Claude、Gemini、Qwen 等自动化开发 agent。假设你没有任何历史上下文，
只通过本文件快速接手 Aureline 项目。

## 项目一句话

Aureline（曜线）是一个 Avalonia 12 + .NET 11 Preview 4 的 Android/iOS 代理客户端壳。
共享 UI 和业务状态在 `Aureline/`，Android/iOS 平台项目负责 VPN、native core 和系统 API。

核心调用链：

```text
Avalonia UI / C#
        |
IClashRuntime
        |
Android VpnService 或 iOS PacketTunnel NetworkExtension
        |
P/Invoke
        |
libclash / clash-rs / libmeow
```

本项目不使用 Kotlin-Dart、gomobile、项目自有 C++ JNI 胶水。Android 系统能力通过
.NET for Android 绑定调用；iOS VPN 逻辑运行在 NetworkExtension 扩展进程中。

## 当前状态

- 项目正式名：`Aureline`，中文名“曜线”。
- 许可证：客户端自有代码 Apache-2.0；包含 Go/mihomo `libclash` 的产物同时有 GPL-3.0 义务。
- Android Release 基线：项目内固定 `.NET 11 Preview 4` + Android NativeAOT。
- iOS 构建：本机不要求支持，统一走 GitHub Actions。
- UI 主线已完成 P0-P4 第一轮重构：
  - P0：设计系统、导航壳、能力模型。
  - P1：总览页。
  - P2：策略页。
  - P3：配置页。
  - P4：工具页。
- 下一阶段重点：真机验收、Release NativeAOT 验证、iOS 共享 UI 验证、细节打磨。

## 关键目录

```text
Aureline/
  共享 Avalonia UI、ViewModel、状态、Clash runtime 抽象。

Aureline.Android/
  默认 Android 变体，使用 Go/mihomo libclash.so。

Aureline.ClashRs.Android/
  clash-rs Android 变体，复用共享 UI 和 Android 主体。

Aureline.Meow.Android/
  meow Android 变体，使用 libmeow.so 和独立 libmeow_* ABI。

Aureline.iOS/
  iOS 主 App，负责 NETunnelProviderManager。

Aureline.iOS.PacketTunnel/
  iOS NetworkExtension，负责 tunnel、utun fd、libclash.a P/Invoke、IPC。

libclash/
  Go/CGO native core submodule，不要当成普通应用代码随意重构。

docs/
  项目结构、native ABI、UI/UX 设计和实施计划。
```

详细结构参考：

- `docs/project-structure.md`
- `docs/native-core-abi.md`
- `docs/ui-ux-design.md`
- `docs/ui-ux-implementation-plan.md`

## 架构约束

1. 共享项目 `Aureline/` 不允许引用 `Aureline.Android/`、`Aureline.iOS/` 或任何平台项目。
2. UI 和 ViewModel 不直接调用 `LibClashNative`，必须通过 `IClashRuntime`。
3. native ABI 差异必须留在对应平台项目的 `Interop/LibClashNative.cs` 内适配。
4. 多 core 能力差异通过 `CoreCapabilities` 表达，不支持的功能不要做成可点击占位。
5. iOS 主 App 不直接运行核心，也不直接 P/Invoke `libclash`；核心只在 PacketTunnel 扩展进程。
6. iOS 默认不启用 `external-controller`；主 App 与扩展通过 `SendProviderMessage` IPC。
7. Android VPN 逻辑只放在 `VpnService` 和 Android runtime，不能塞进共享 UI。
8. `libclash/` 是 git submodule。除非任务明确要求，不要在主仓库任务中重写 submodule。
9. `Aureline.Android/Interop/LibClashNative.cs`、clash-rs ABI、meow ABI 不要求完全一致。
10. 不要把 mihomo、Clash、MetaCubeX、clash-rs、meow 名称作为应用品牌。

## UI/UX 约束

项目已经放弃 1:1 复刻 FlClash，转向 Aureline 自有 UI 语言“曜线”。

必须遵守：

- 不复刻 FlClash 的页面布局、组件外观、图标、动画或配色。
- 不做营销式首页，第一屏就是可操作的运行控制台。
- 不做假按钮、假菜单、假仪表盘、假配置项。
- 所有可见入口必须真实可执行，或按 capability 隐藏/禁用并给出明确原因。
- 页面职责：
  - 总览：连接状态、实时流量、网络检测、内网地址、最近事件。
  - 策略：出站模式、策略组、节点选择、测速。
  - 配置：profile、订阅、本地配置导入、启用/更新/校验/删除。
  - 工具：应用、核心、数据、诊断。
- 配置页不放运行参数；运行参数放工具页。
- 工具页不得出现无业务的 `advanced/rules/scripts` 占位入口。
- 卡片圆角不超过 8px，避免卡片套卡片。
- 使用 `Aureline/Styles/Palette.axaml` 的语义色，不在页面里随意写死颜色。
- 长文本必须适配移动端，不允许挤压、重叠或越界。

当前 UI 设计文档在 `docs/ui-ux-design.md`。

## ViewModel 和状态约束

主界面 ViewModel 是 partial 拆分：

```text
Aureline/ViewModels/Main/MainViewModel.cs
Aureline/ViewModels/Main/MainViewModel.Runtime.cs
Aureline/ViewModels/Main/MainViewModel.Config.cs
Aureline/ViewModels/Main/MainViewModel.Proxies.cs
Aureline/ViewModels/Main/MainViewModel.State.cs
Aureline/ViewModels/Main/MainViewModel.Tools.cs
```

开发时按职责归位：

- 启动/停止/VPN profile：`MainViewModel.Runtime.cs`
- 配置文件/订阅/校验：`MainViewModel.Config.cs`
- 策略组/节点/测速/流量：`MainViewModel.Proxies.cs`
- SQLite 持久化和 profile 列表：`MainViewModel.State.cs`
- 工具页设置、备份恢复、访问控制：`MainViewModel.Tools.cs`
- 页面导航、共享派生属性：`MainViewModel.cs`

状态已接入 SQLite。不要把持久化状态重新改回纯内存或散落到平台层。

## Android 约束

默认 Android 项目：

```text
Aureline.Android          com.embermoth.aureline
Aureline.ClashRs.Android  com.embermoth.aureline.clashrs
Aureline.Meow.Android     com.embermoth.aureline.meow
```

Android 运行链路：

1. UI 调用 `IClashRuntime.StartAsync`。
2. Android runtime 初始化 native core。
3. `VpnService` 获取 TUN fd。
4. fd 传给 native core 的 `*_start_tun`。
5. socket protect、UID 查询通过 reverse P/Invoke 回调。

约束：

- 不要在共享 UI 中直接操作 Android `VpnService`。
- 不要混入 Kotlin/JNI 胶水。
- Release 构建使用 NativeAOT。
- Android 16 需要注意 native so 的 16 KB page size。
- 本仓库不提交变体 native `.so` 产物，除非任务明确要求。

## iOS 约束

iOS 必须使用 NetworkExtension：

- `Aureline.iOS/` 是主 App，只负责 `NETunnelProviderManager`。
- `Aureline.iOS.PacketTunnel/` 是扩展进程，负责设置 tunnel、扫描 `utun` fd、调用 `libclash.a`。
- 主 App 和扩展用 `NETunnelProviderSession.SendProviderMessage` 做 IPC。
- 扩展内存预算更紧，停止/启动后会触发 .NET GC 和 `libclash_force_gc`。

禁止：

- 不要让 iOS 主 App 直接 P/Invoke `libclash`。
- 不要把 iOS 常规控制面改成 `external-controller` REST fallback。
- 不要假设 iOS 和 Android 可以共享同一个进程模型。

## Native ABI 约束

默认 ABI 文档见 `docs/native-core-abi.md`。

新增 ABI 时：

1. 先更新对应 native core。
2. 再更新平台项目 `Interop/LibClashNative.cs`。
3. 再通过 `IClashRuntime` 暴露共享能力。
4. 最后更新 `docs/native-core-abi.md`。

不要让页面直接依赖某个 native 函数名。

## 构建命令

初始化：

```bash
git submodule update --init --recursive
./scripts/install-dotnet11-sdk.sh
./scripts/install-dotnet11-android-minimal.sh
```

默认 Android Debug：

```bash
./scripts/dotnet11.sh build Aureline.Android/Aureline.Android.csproj \
  -c Debug \
  -f net11.0-android \
  -r android-arm64 \
  -v:minimal
```

clash-rs 变体 Debug：

```bash
./scripts/dotnet11.sh build Aureline.ClashRs.Android/Aureline.ClashRs.Android.csproj \
  -c Debug \
  -f net11.0-android \
  -r android-arm64 \
  -v:minimal
```

meow 变体 Debug：

```bash
./scripts/dotnet11.sh build Aureline.Meow.Android/Aureline.Meow.Android.csproj \
  -c Debug \
  -f net11.0-android \
  -r android-arm64 \
  -v:minimal
```

Android Release NativeAOT：

```bash
./scripts/publish-android-nativeaot.sh android-arm64
```

当前 .NET 11 Preview 4 Android NativeAOT runtime pack 主要覆盖 `android-arm64` 和
`android-x64`。不要承诺 armv7 NativeAOT release，除非 runtime pack 已确认支持。

## 验证要求

做代码改动后，至少运行对应 Debug 构建。UI 或运行时改动优先跑默认 Android：

```bash
./scripts/dotnet11.sh build Aureline.Android/Aureline.Android.csproj \
  -c Debug \
  -f net11.0-android \
  -r android-arm64 \
  -v:minimal
```

涉及 release、trim、AOT、P/Invoke、native library 的改动，必须追加：

```bash
./scripts/publish-android-nativeaot.sh android-arm64
```

真机检查优先项：

- App 启动不闪退。
- 底部导航居中。
- Android 返回键先返回 UI 层级，不直接退出。
- 启动/停止 VPN 可用。
- 配置导入后能启用并重启核心。
- 策略组和节点切换能生效。
- 深色/浅色模式不出现文本重叠或不可读。

## Git 和工作区规则

- 工作区可能已有用户或其他 agent 的变更，先用 `git status --short` 看清楚。
- 不要随意 `git reset`、`git checkout --`、删除文件或清空暂存区。
- 不要回滚你没有创建的改动。
- 用户没要求提交时，不要自动 commit。
- 如果当前存在 staged 变更，不要擅自整理暂存区。
- 修改 submodule 前必须确认任务确实要求改 submodule。

## 常见坑

- `ProfilesPageView` 是旧页面，当前主导航已经使用 `ConfigPageView` 管 profile/订阅。
- 工具页不能重新加 `advanced/rules/scripts` 这种无业务占位。
- iOS 和 Android 是不同进程模型，不要照搬 Android runtime 到 iOS 主 App。
- `external-controller` 会增加内存和控制面暴露，iOS 默认不用。
- URL 订阅导入必须使用 `User-Agent: clash`。
- `Geo低内存模式` 对应 YAML `geodata-loader: memconservative`。
- Rust core 的内存控制 API 可以是占位，但 UI 不能假装支持 Go GC 行为。
- NativeAOT/trim 警告不能简单忽略，尤其是 P/Invoke、序列化、反射相关。

## 推荐接手流程

1. 读 `README.md`、本文件、`docs/project-structure.md`。
2. 看 `git status --short`，确认当前 staged/unstaged 状态。
3. 如果改 UI，读 `docs/ui-ux-design.md` 和 `docs/ui-ux-implementation-plan.md`。
4. 如果改 native ABI，读 `docs/native-core-abi.md`。
5. 小范围修改，优先沿用现有 partial ViewModel 和样式 token。
6. 构建验证。
7. 最终说明改了什么、验证结果、还剩什么风险。

---
> Source: [Ember-Moth/Aureline](https://github.com/Ember-Moth/Aureline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
