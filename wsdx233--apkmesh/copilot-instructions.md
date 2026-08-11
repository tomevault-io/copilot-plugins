## apkmesh

> 本文件适用于仓库根目录及其所有子目录。父目录中的 `/home/wsdx233/AGENTS.md` 也适用；规则冲突时，以路径更具体的规则为准。

# APK Mesh 代理开发指南

本文件适用于仓库根目录及其所有子目录。父目录中的 `/home/wsdx233/AGENTS.md` 也适用；规则冲突时，以路径更具体的规则为准。

## 项目概览

APK Mesh 是一个 Flutter 多平台 APK 源聚合客户端。应用 UI 使用 Dart/Flutter 实现，源适配器是由 QuickJS 执行的独立 JavaScript 文件。JavaScript 不能直接访问网络、文件系统或系统安装器，而是通过宿主提供的受权限策略约束的 `apkmesh.*` API 工作。

当前包要求 Dart SDK `^3.12.2`，Flutter 工具链使用 stable channel。Android 是具备完整 QuickJS、隐藏 WebView、下载和 APK 安装能力的主要运行目标；其他平台保留不同程度的演示或受限实现。

## 目录结构

```text
.
├── lib/
│   ├── main.dart                    # 仅负责 Flutter 入口
│   ├── app.dart                     # 应用生命周期、主题和一级导航 Shell
│   ├── pages/                       # 按用户流程拆分的页面和复杂弹层
│   │   ├── home_page.dart           # 首页、搜索、分页和分类内容
│   │   ├── downloads_page.dart      # 下载列表、控制和安装动作
│   │   ├── sources_page.dart        # 源管理和源导入
│   │   ├── settings_page.dart       # 设置和翻译设置
│   │   ├── debug_sheet.dart         # 调试面板及其详情视图
│   │   └── details_sheet.dart       # 应用详情、源匹配和下载候选
│   ├── core/                        # 状态、模型、源运行时和平台能力
│   │   ├── app_state.dart           # 全局状态、源生命周期、搜索和下载编排
│   │   ├── models.dart              # 源、应用、分类、调试、下载数据模型
│   │   ├── source_runtime.dart      # SourceHostApi、源接口、权限策略、SourceRegistry
│   │   ├── quickjs_source*.dart     # QuickJS 条件导入门面和平台实现
│   │   ├── host_factory*.dart       # HTTP、WebView、下载、安装宿主
│   │   ├── debug_log.dart           # 请求、WebView 和运行日志存储
│   │   └── download_* / translation_service.dart # 下载持久化、通知和翻译
│   └── widgets/                     # 跨页面复用的展示组件
├── assets/sources/                   # Source API v1 的独立 JavaScript 源脚本
├── docs/source-api.md                # JavaScript 源契约和宿主能力文档
├── test/                             # Flutter 单元测试、Widget 测试和运行时测试
├── tools/source_debugger/            # 独立 Python QuickJS/HTTP/Playwright 调试器
│   ├── src/apkmesh_debug/            # CLI、运行时、策略、HTTP、浏览器、回放和 trace
│   ├── tests/                        # 调试器通用测试
│   ├── examples/                     # 示例源、回放 fixture 和目录检查脚本
│   ├── fixtures/                     # 可提交的示例回放定义
│   ├── pyproject.toml                # Python 项目和 pytest 配置
│   └── uv.lock                       # Python 依赖锁文件
├── android/                          # Android Gradle 工程和 Kotlin 原生桥接
├── ios/ macos/ linux/ web/ windows/  # Flutter 各平台壳工程
├── pubspec.yaml / pubspec.lock       # Dart/Flutter 依赖和锁文件
├── analysis_options.yaml             # Flutter lint 配置
├── README.md                         # 用户说明、开发入口和法律边界
└── LICENSE
```

平台目录中由 Flutter 或插件生成的文件（例如 `build/`、`.dart_tool/`、`linux/flutter/`、`windows/flutter/` 中的生成注册文件）不是业务逻辑。除非任务明确涉及平台工程，否则不要手工修改或提交生成产物。

### Dart/Flutter 模块边界

- `main.dart` 只保留 `main()` 和入口导入；主题、生命周期和一级导航放在 `app.dart`。
- `pages/` 按用户流程组织页面。页面可以组合 `core` 状态和 `widgets` 展示组件，但网络解析、权限校验和下载编排必须留在 `core` 或源脚本边界。
- `widgets/` 只放跨页面复用的展示组件或局部交互组件；例如下载状态控件和截图查看器不得重新复制到各页面。
- 新增跨文件使用的 Widget 或 helper 才公开命名；仅在所属模块内使用的类型保持私有，避免无意扩大 API。公开 Widget 构造函数必须保留 `key`。
- 单文件超过约 1000 行时，优先按完整职责提取到独立模块，并同步迁移测试和 import；不要用机械切片或为了降低行数引入无语义的包装层。`core` 中的状态编排可保持集中，但新增功能应优先放入独立服务或运行时模块。
- `core` 不得反向依赖 `pages/` 或 `widgets/`；入口层可以依赖页面层，页面层可以依赖核心层。

`lib/features/` 目前不是实际的业务代码目录；新增页面应放在 `lib/pages/`，只有在拆分能降低真实复杂度时才增加新的功能子目录。

## 运行时架构

主调用链如下：

```text
Flutter UI (`app.dart` + `pages/`)
        |
     AppState
        |
   SourceRegistry  <-- ApkSourceScript / SourceCatalogScript / DebugProjectSource
        |
QuickJsApkSourceScript (Android) or built-in Demo source
        |
SourceHostApi
        |
NativeHostApi (HTTP + Headless WebView + download/install)
       or DemoHostApi (受限/不可用的演示实现)
```

- `main.dart` 的 `Shell` 已迁移到 `app.dart`，提供主页、下载、源管理、设置四个一级入口；详情、调试和截图查看器分别位于 `pages/` 与 `widgets/`，仍以 sheet/dialog 形式打开。
- `AppState.initialize()` 当前在 Android 上通过 Flutter `AssetManifest` 自动扫描并加载 `assets/sources/` 下的所有 `.js` 源脚本，源管理元数据来自各脚本的 manifest；不支持 QuickJS 的平台保留内置演示源。新增源脚本只要纳入 `assets/sources/` 资产即可被发现。
- `SourceRegistry.search()` 并发调用启用源，保留单个源错误并聚合其他成功结果；修改此行为时必须同步更新测试和 UI 的错误展示逻辑。
- `quickjs_source_io.dart` 目前只在 Android 创建 QuickJS 源运行时；`quickjs_source_stub.dart` 在不支持平台返回空实现。`host_factory_io.dart` 的隐藏 WebView 支持 Android/iOS/macOS，APK 安装仅支持 Android；Linux/Windows 没有隐藏 WebView 实现，Web 使用 stub 宿主。
- Android 原生 `MainActivity.kt` 通过 `com.apkmesh/install` 和 `com.apkmesh/download_notifications` 两个 MethodChannel 提供安装权限检查、未知来源权限跳转和下载通知。
- `tools/source_debugger` 是与 Flutter 宿主平行的 Python 实现，用于无 Android 设备执行源脚本，并支持 live、record、replay 三种模式。它不是 Flutter 应用的运行时依赖。

## Source API 约束

Source API 的规范来源是 [`docs/source-api.md`](docs/source-api.md)。每个源是单个可独立加载的 JavaScript 文件，必须设置 `globalThis.source`，并提供：

- `manifest`：至少包含源 ID、名称、版本、主页和权限声明；
- `async search(query, page)`：返回应用摘要数组；
- `async details(idOrUrl)`：返回完整应用详情和下载项；
- 可选的 `home()`、`category(categoryId)` 和 `debug(projectId, input)`。

源返回的数据必须符合文档中的字段形状。新增或修改以下任一契约时，必须成组检查并按需更新：

1. `docs/source-api.md`；
2. Dart 的 `SourceHostApi`、模型、QuickJS bootstrap/消息桥和宿主实现；
3. Python 调试器的 bootstrap、dispatch、模型和宿主实现；
4. 相关 JavaScript 源、回放 fixture 及 Dart/Python 测试。

不要只修改一个端的桥接代码。尤其要确认 `headers`、选择器查询结果、异步 Promise、下载和错误的序列化在 Flutter QuickJS 与 Python QuickJS 中保持一致。

## 安全与法律边界

这些规则是功能约束，不是可选的产品建议：

- 所有源网络、浏览器、下载和安装操作都必须通过 `SourcePolicy` 声明的权限；不得在 UI 或源脚本中绕过宿主检查。
- 网络规则使用 manifest 中的精确主机名、`*.example.com` 子域名规则或显式的 `'*'` 任意主机权限。`'*'` 只允许 `http`/`https`，适用于用户明确信任且会返回临时下载主机的源；每一次 HTTP 重定向仍要重新校验协议和 manifest 权限。
- 浏览器导航和资源拦截必须继续遵守源白名单。源打开标签页后要在 `finally` 中关闭，避免 Headless WebView 和控制器泄漏。
- `apkmesh.install()` 不能静默安装。必须保留用户确认、Android 未知来源权限检查和平台能力检查；调试器中的安装能力明确不可用。
- 不新增任意文件读写、系统命令、Cookie 导出、账号凭据收集或后台安装能力。不要把密钥、Cookie、真实下载内容或包含敏感信息的网络录制提交到仓库。
- 只实现有权访问和使用的站点源，不实现绕过访问控制、窃取账号、解析明显侵权内容或传播恶意软件的逻辑。第三方源的条款、版权、文件签名和安全性必须由使用者自行确认。
- 对下载直链和中转链路要进行协议、来源和 manifest 权限校验。源可以通过 `network: ['*']` 显式声明任意主机，但这必须是源的明确权限选择，下载内容、文件签名和第三方源安全性由使用者自行确认。

## Dart/Flutter 开发约束

- 遵循 `analysis_options.yaml` 中的 `flutter_lints`，提交前运行 `dart format` 和 `flutter analyze`。不要用大范围格式化改动无关文件。
- 优先复用现有模型和接口；状态变更通过 `AppState`/`ChangeNotifier` 暴露，列表和 map 对外保持不可变视图。异步 UI 更新要检查 `mounted`，并释放 controller、timer、WebView、HTTP client 和 QuickJS runtime。
- 保持条件导入的两端都能编译：修改 `*_io.dart` 时检查对应的 `*_stub.dart` 接口是否仍完整。Web 和桌面测试不能假设 Android 的安装、通知或隐藏 WebView 能力存在。
- 不把网络解析逻辑塞进 Widget。源解析应位于 JavaScript 源或宿主/运行时边界；UI 只负责调用状态层并展示加载、空结果和部分失败状态。
- 保留源搜索的并发语义和单源容错，除非需求明确要求改变聚合行为。新增异步流程时覆盖成功、失败、取消、重复调用和释放后的状态。
- 修改 Android MethodChannel 时，Dart 方法名、参数键、Kotlin handler、Manifest 权限和平台版本分支必须一起检查；必要时增加原生手动验证，不以 Widget 测试替代。
- 依赖变更必须同时审阅 `pubspec.yaml` 与已跟踪的 `pubspec.lock`，并说明新增依赖的必要性。不要直接编辑插件生成文件来修复构建问题。

## JavaScript 源开发约束

- 源脚本必须是可独立执行的单文件，不依赖 Node、浏览器全局 DOM 或未声明的模块；页面 DOM 操作通过 `apkmesh.browser`，普通 HTTP 通过 `apkmesh.request`。
- 在 manifest 中只声明确实需要的主机和能力；网络权限可使用精确主机名、`*.example.com` 或在确有临时下载主机需求时使用 `'*'`。使用 `https`、规范化 URL，并在解析详情、截图和下载项时过滤空值、重复值和不可信域名。
- 对动态页面优先使用 `browser.open`/`waitFor`/`query`/`queryAll`；无论成功还是异常都关闭标签页。解析懒加载图片时同时考虑站点实际使用的 `src`、`data-src`、`data-lazy-src` 等属性。
- 保持 `search()`、`details()` 返回字段与 Source API 一致；站点改版时不要用静默的默认值掩盖解析失败，应让调试结果或源错误可观察。
- 新增源时不要默认加入应用。若确实要内置，需同时更新 `AppState` 的源元数据和 Android 加载列表，并说明其权限、法律依据和失败行为；否则先用调试器验证。
- 线上站点响应可能变化或受 Cloudflare 等验证影响。为可重复测试优先使用 record 生成、人工审阅后提交的 replay fixture；不要在自动化测试中依赖实时第三方网络。

## Python 调试器约束

在 `tools/source_debugger/` 内使用 Python `>=3.12` 和 `uv` 管理环境：

```bash
cd tools/source_debugger
uv sync
uv run playwright install chromium  # 首次使用或浏览器版本变化时
uv run pytest tests --ignore=tests/sources
```

常用源检查命令：

```bash
uv run apkmesh-debug ../../assets/sources/apkvision.js inspect
uv run apkmesh-debug --mode replay --fixture-dir examples/fixtures/demo \
  examples/fixture_demo.js search demo
uv run python examples/check_catalog.py ../../assets/sources/apkvision.js
```

- `live` 可访问网络，`record` 保存响应和 DOM 快照，`replay` 必须完全依赖 `replay.json`；回放中未登记的 URL 必须失败，不能偷偷回退到网络。
- fixture 的 URL 必须精确匹配请求（包括查询参数），响应体优先使用 `body_file`，并检查录制内容不含凭据或不必要的大型二进制文件。
- 通用宿主测试放在 `tools/source_debugger/tests/` 根目录。`tests/sources/`、`.venv/`、缓存、trace 和下载目录由 `.gitignore` 忽略，除非明确决定将某项测试/fixture 纳入版本控制，否则不要提交它们。
- Python 侧的权限检查、重定向、QuickJS 消息名和 Flutter 侧保持一致；修改一侧时必须运行对应的通用测试并检查另一侧契约。

## 验证清单

按改动范围执行最小但完整的检查：

```bash
# Flutter/Dart
flutter pub get
dart format lib test
flutter analyze
flutter test

# Python 调试器
cd tools/source_debugger
uv run pytest tests --ignore=tests/sources
```

涉及源脚本或 Source API 时，还要运行对应源的 `inspect`/离线 `replay` 检查；涉及 Android 宿主、通知、安装、下载或隐藏 WebView 时，要在 Android 模拟器或真机进行手动冒烟测试，并记录未覆盖的平台限制。完成任务时报告实际执行过的命令和任何未验证项。

## 工作区与提交纪律

- 开始修改前查看 `git status`，保留用户已有的改动；不要用 `git reset --hard`、`git checkout --` 或批量删除来清理工作区。
- 只修改完成任务所需的文件。不要把 `build/`、`.dart_tool/`、平台生成目录、Python 虚拟环境、缓存、实时 trace 或下载文件加入提交。
- 未经明确要求不要删除源、fixture、平台文件或文档。重构应与功能改动分开，避免无关的元数据和格式化噪声。
- 代码、测试、接口文档和安全边界需要一起维护；如果当前实现与文档不一致，应在开发报告中明确指出，并优先修正会造成安全或数据损坏的差异。

---
> Source: [wsdx233/ApkMesh](https://github.com/wsdx233/ApkMesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
