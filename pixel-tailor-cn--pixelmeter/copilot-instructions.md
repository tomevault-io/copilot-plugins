## pixelmeter

> 本文件是本仓库的统一协作规范。所有智能体、自动化助手和开发者在处理本仓库任务时，均应优先遵循本文件。

# AGENTS.md

本文件是本仓库的统一协作规范。所有智能体、自动化助手和开发者在处理本仓库任务时，均应优先遵循本文件。

## 总则

- 使用中文回复用户。
- 生成的代码注释和文档使用中文。
- 日志打印使用英文。
- 涉及专业术语（如 Interface、Tun、Binder、Overlay、StateFlow、Repository）时保留英文，必要时附带中文解释。
- 当需求不明确时，先提问澄清，尤其是网络接口过滤规则、悬浮窗交互、后台保活等容易受系统限制影响的需求。
- 对用户提出的方案，需要结合 Android 系统限制进行分析，特别是后台限制、前台服务限制、通知权限和悬浮窗权限。

## 项目概述

PixelMeter 是一款面向 Google Pixel 及原生 Android 设备的网速监控应用。核心特性是通过
`ConnectivityManager` + `TrafficStats` 过滤 VPN 虚拟接口（如 `tun0`
），仅统计物理网络接口（Wi-Fi、Cellular、Ethernet）的流量，避免 VPN 场景下流量双重计数。

## 构建命令

```bash
# Debug 构建
./gradlew :app:assembleDebug

# Release 构建（需要在 local.properties 中配置签名信息）
./gradlew :app:assembleRelease

# Lint 检查
./gradlew lint
```

项目不需要编写单元测试或 Android 测试，验证以构建、Lint 和真机功能验证为主。

## SDK 与工具链

- **MinSDK**: 31 (Android 12)
- **CompileSDK/TargetSDK**: 37
- **Kotlin**: 2.4.0
- **JVM Target**: 21
- **AGP**: 9.2.1
- **Compose BOM**: 2026.05.01
- 全局 opt-in：`ExperimentalMaterial3Api`
- 版本目录：`gradle/libs.versions.toml`
- 版本号策略：`versionCode` 来自 git commit 数量，`versionName` 来自 `libs.versions.toml` 中的
  `app-version`
- 支持语言：`en`、`zh-rCN`

## 架构

项目是单模块应用（`app/`），采用 **MVVM** 架构，使用 Kotlin + Jetpack Compose (Material 3)，依赖注入使用
**Koin**。

包根路径：`vip.mystery0.pixel.meter`，位于 `app/src/main/kotlin/`。

### 数据流

```text
SpeedDataSource (ISpeedDataSource)
  ↓ 通过 TrafficStats 逐接口读取物理网卡流量
NetworkRepository
  ↓ 定时轮询数据源，计算速率差值 → StateFlow<NetSpeedData>
  ↓ 从 DataStoreRepository 同步所有用户偏好设置为 StateFlow
NetworkMonitorService (前台服务)
  ↓ 收集 netSpeed flow
  ├→ NotificationHelper：将网速渲染为动态 Bitmap 图标或 Live Update 文本
  └→ OverlayWindow：基于 Compose 的悬浮窗，通过 WindowManager 挂载
```

### 核心组件

- **`SpeedDataSource`** (`data/source/impl/`)：注册 `NetworkCallback`，过滤 Wi-Fi、Cellular、Ethernet
  物理接口，用 `ConcurrentHashMap` 缓存接口名，调用 `TrafficStats.getRxBytes/getTxBytes` 逐接口读取流量。这是
  VPN 流量过滤的核心。
- **`NetworkRepository`** (`data/repository/`)：中央状态枢纽。按用户配置的采样间隔轮询数据源计算速率差值，暴露多个
  `StateFlow` 属性（镜像自 `DataStoreRepository`）。所有设置写入委托给 `DataStoreRepository`。
- **`DataStoreRepository`** (`data/repository/`)：Jetpack DataStore Preferences 封装层。DataStore 名称：
  `pixel_pulse_preferences`。
- **`NetworkMonitorService`** (`service/`)：前台服务（Manifest 声明 `specialUse|dataSync`，Android 14+
  运行时使用 `FOREGROUND_SERVICE_TYPE_SPECIAL_USE`，更低版本使用 `DATA_SYNC`）。收集网速
  flow，更新通知和悬浮窗。监听息屏/亮屏事件，息屏 2 分钟后暂停监控以省电。
- **`NotificationHelper`** (`service/`)：构建通知，支持实时 Bitmap 小图标（Canvas 绘制总网速文字）或
  Live Update（`setShortCriticalText` + `setRequestPromotedOngoing`）。
- **`OverlayWindow`** (`ui/overlay/`)：基于 Compose 的悬浮窗，挂载到 `WindowManager`。实现
  `LifecycleOwner`、`ViewModelStoreOwner`、`SavedStateRegistryOwner` 以在 Activity 外宿主 Compose。
- **`AppModule`** (`di/`)：单一 Koin 模块，注册所有依赖。
- **Tile Services** (`service/tile/`)：Quick Settings 快捷磁贴，用于开关通知和悬浮窗。
- **`BootReceiver`** (`receiver/`)：开机自启服务（用户启用时生效）。

### Activities

- `MainActivity`：主界面仪表盘。
- `SettingsActivity`：设置页面，使用 `me.zhanghai.compose.preference` 库。

## 代码质量与开发原则

### 质量要求

- 严格遵循 Kotlin 官方编码规范。
- 遵循 Modern Android Development (MAD) 指南。
- 全面使用 Jetpack Compose、Material 3 和动态取色，支持 Pixel 设备的系统色彩体验。
- 使用 MVVM（ViewModel + StateFlow + Repository）。
- 使用 **Koin**（Koin-Android、Koin-Compose）进行依赖注入。
- 核心功能严格禁止使用 Root 或 Shizuku 权限。
- 单个文件原则上不超过 1000 行。
- 避免在 Activity 中编写业务逻辑。

### 功能特性规范

- 网速统计使用 `TrafficStats` 配合 `ConnectivityManager`。
- 通过 `ConnectivityManager` 遍历物理网络接口（Wi-Fi、Cellular、Ethernet），排除 VPN 虚拟接口，避免流量双重统计，并直接读取
  `TrafficStats` 数据。
- 首次启动不开启显示，由用户选择是否启用。
- 通知栏动态图标（Notification Icon）需要实时绘制 Bitmap；双向模式下合并展示总量。
- 悬浮窗（Floating Window）使用 Compose 挂载到 `WindowManager`，并支持独立开关。
- 快速设置（Quick Settings）通过 `TileService` 支持系统下拉栏快捷开关。

### Service 分离

- 网速监听核心逻辑需运行在前台服务（Foreground Service，type=`dataSync`）中。
- 业务逻辑应放在前台服务和 Repository 等合适层级中，不应堆叠在 Activity 中。

### 注释

- 使用中文编写清晰的 KDoc 与行内注释。
- 核心算法逻辑（如网络接口过滤）必须添加详细注释说明。

## 测试与验证

- 本项目不做单元测试：除非用户明确要求，不新增、修改或维护单元测试代码。
- 此项目不需要编写单元测试或 Android 测试。
- 变更代码后，必须执行 `./gradlew :app:assembleDebug` 确保编译通过。
- 运行 `./gradlew lint` 检查潜在的代码质量问题。
- 重点测试开启 VPN 场景下的网速统计是否准确，不应包含 VPN 虚拟网卡流量。
- 必须在真机上测试，优先使用 Pixel 设备。

### 项目本地校验流程

1. 命令行执行 `./gradlew :app:assembleDebug`。
2. 确认无编译错误，且 `libs.versions.toml` 中无过时警告。
3. 根据变更范围执行 `./gradlew lint`。

## 文档与记忆

文档与记忆采用 Markdown 格式，存放于 `.agentdocs/` 及其子目录下。

索引文档：`.agentdocs/index.md`

### 文档分类

- `prd/`：产品与需求。
    - `prd/requirements.md`：核心功能需求（PixelMeter 功能清单）。
- `architecture/`：架构与技术细节。
    - `architecture/data-source-strategy.md`：单一数据源策略（TrafficStats + ConnectivityManager）。
    - `architecture/service-lifecycle.md`：前台服务保活、`specialUse|dataSync` 类型与 Android 14+ 适配。
- `ui/`：界面规范。
    - `ui/design-system.md`：Material 3 主题与悬浮窗设计规范。
- `workflow/`：任务流文档，按标准格式命名。

### 全局重要记忆

- **项目名称**：PixelMeter。
- **设备支持**：
    - 核心目标：Google Pixel 系列。
    - 兼容目标：运行原生或类原生 Android（AOSP）的设备。
- **关键技术决策**：
    - **DI**：Koin，轻量级，适合本工具。
    - **Settings UI**：`me.zhanghai.compose.preference` + `com.github.skydoves:colorpicker-compose`。
    - **Browser**：Chrome Custom Tabs (CCT)，用于集成 Cloudflare 测速。

## 任务处理指南

- 需求不明确时先澄清，例如具体的过滤接口名称、悬浮窗吸附逻辑、通知呈现方式等。
- 涉及系统底层（如读取 `/proc`）或保活策略修改时，需要在文档中记录潜在兼容性风险。
- 分阶段实施复杂任务：
    1. 基础架构搭建（Koin、Compose）。
    2. 网速监听实现（TrafficStats + ConnectivityManager）。
    3. UI 层实现（通知栏绘图、悬浮窗）。
    4. CCT 集成与设置页完善。

### 任务回顾

在任务完成并呈现最终消息前，必须进行以下回顾：

- 检查是否产生新的可复用组件，并按需更新架构文档。
- 检查 `.agentdocs/` 下的文档是否需要更新。
- 确认 `libs.versions.toml` 中的依赖版本是否需要关注最新稳定版。

## 签名配置

Release 签名从 `local.properties` 读取，键名如下：

- `SIGN_KEY_STORE_FILE`
- `SIGN_KEY_STORE_PASSWORD`
- `SIGN_KEY_ALIAS`
- `SIGN_KEY_PASSWORD`

如果 `local.properties` 不可读，则回退到同名环境变量。

---
> Source: [Pixel-Tailor-CN/PixelMeter](https://github.com/Pixel-Tailor-CN/PixelMeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
