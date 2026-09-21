## ai-offline-translator

> - **语言 / 框架** — Dart + Flutter（v3.9.2 SDK），目标平台 macOS & Android

# Repository Guidelines

## 技术栈

- **语言 / 框架** — Dart + Flutter（v3.9.2 SDK），目标平台 macOS & Android
- **推理引擎** — llama.cpp（C++ 静态库，PR #22836 提供 STQ1_0 格式支持）
- **原生桥接** — `translator_engine.hpp`（共享 C++ header）+ ObjC++（macOS）/ JNI（Android）
- **主要依赖** — `file_picker`（模型文件导入）、`url_launcher`（模型下载）、`flutter_lints`（lint 规则）、`cupertino_icons`

## 项目结构与模块组织

| 路径 | 内容 |
| --- | --- |
| `flutter_app/` | Flutter 项目根 — UI（Dart）、测试、平台 runner、原生桥接 |
| `flutter_app/lib/` | Dart 源码 — `features/translator/`（主页面/controller/channel）、`design/`（主题/颜色/间距）、`about/` |
| `flutter_app/native/translator_engine/` | 共享 C++ 推理引擎 — `translator_engine.hpp` + `.cpp` |
| `flutter_app/test/` | Flutter 测试 — widget test + `features/` 下按功能分组的单元测试 |
| `third_party/llama.cpp/` | Git submodule，固定到 PR #22836 commit |
| `scripts/` | 构建与 smoke-test 脚本 — `setup.sh` / `setup.ps1`、`build_android_llama.sh`、`safe_llama_smoketest.py` / `.ps1` |
| `models/` | 模型参考文档 + `.gitignore` 排除的二进制目录（GGUF 文件不入 Git） |
| `docs/` | GitHub Pages 介绍页（`index.html`）、内部设计文档（`internal/`）、计划（`superpowers/`） |
| `DESIGN.md` | 视觉设计系统（MiniMax 风格的颜色、字体、间距、按钮、卡片） |

`docs/` 下的关键文档：

- `docs/models_inventory.md`：本机模型清单与兼容性记录。
- `docs/llama_pr22836_notes.md`：`llama.cpp` PR #22836 与 `STQ1_0` 加载说明。
- `docs/index.html`：GitHub Pages 介绍页（发布页），展示项目特性、架构、下载入口和 Demo 截图，截图素材在 `docs/public/`。
- `docs/flutter_mobile_architecture.md`：Flutter UI 与原生推理层架构。

`third_party/llama.cpp/` 是 Git submodule，固定到 `llama.cpp` `PR #22836` 对应 commit。

## 构建与开发命令

在 `flutter_app/` 下执行（除非另注）：

| 操作 | 命令 |
| --- | --- |
| 安装依赖 | `flutter pub get` |
| 静态分析 | `flutter analyze` |
| 运行测试 | `flutter test` |
| 运行 App | `flutter run -d <device>`（macOS / Android） |
| 构建 APK | `flutter build apk --target-platform android-arm64 --release` |
| 构建 macOS DMG | `flutter build macos --release` |
| Python 脚本 | `uv run scripts/safe_llama_smoketest.py`（禁止裸 `python3`） |
| llama.cpp submodule | `git submodule update --init third_party/llama.cpp` |

所有 Python 操作必须通过 `uv` 执行，例如 `uv run`、`uv add` 或 `uv sync`。即使只使用标准库的内联脚本（如一行的 `uv run python -c "..."`），也必须走 `uv run`，禁止直接调用系统 `python3`。

第三方依赖使用 Git submodule 管理：

- `git submodule update --init third_party/llama.cpp`：初始化 `llama.cpp` submodule。
- `git submodule status`：确认当前 submodule 指针。
- 不要把 `third_party/llama.cpp/` 改回临时 clone；如需更新 PR 指针，应更新 submodule commit 并同步修改 README / setup 脚本中的校验 commit。

### llama.cpp 安全执行规则

- 在 Windows 上，**禁止直接运行交互式** `llama-cli` 做手工验证，优先使用受限脚本：`uv run scripts/safe_llama_smoketest.py`。
- Windows 上默认优先 GPU offload；如果没有明确确认 GPU 可用，禁止运行高负载推理命令。
- 未经明确授权，不要在 Windows 上执行 CPU-only 的长时间模型推理；如必须回退 CPU，只能使用受限参数和超时控制。
- 所有 smoke test 必须满足：非交互、可超时、有限输出、有限线程、有限 `n_ctx`、有限 `n_predict`。
- 新增推理脚本时，必须优先考虑"失败时自动退出、超时 kill、输出上限、资源上限"，避免拖死主机。
- `third_party/llama.cpp` 必须保持支持 `STQ1_0` 的 PR #22836 兼容路径；更新 submodule 前必须先验证 `Hy-MT1.5-1.8B-STQ1_0.gguf` 可加载并能完成最小翻译。

## 模型兼容性与下载源

### 铁律：未在目标设备验证可加载+可翻译的模型，禁止写入 `supported_model_info.dart`

`supported_model_info.dart` 中列出的模型会直接展示给用户下载。如果模型无法加载或推理乱码，用户体验直接崩溃。**任何新增模型必须先在真机上验证：能加载 + 能完成至少一次正确翻译。**

### 当前模型兼容性实测（2026-07-30 Android 真机验证）

| 模型文件 | 加载 | 翻译质量 | 结论 |
| --- | --- | --- | --- |
| `Hy-MT1.5-1.8B-STQ1_0.gguf` | ✅ | 偶发错误 token（STQ 量化精度损失），但可用 | **唯一可用模型** |
| `Hy-MT1.5-1.8B-1.25bit.gguf`（非 STQ） | ❌ 加载失败 | — | 引擎不支持其量化类型 |
| `Hy-MT2-1.8B-1.25Bit.gguf` | ✅ | ❌ 全乱码（如 Hello→中文 输出 `ㄇ'`） | STQ 路径与 Hy-MT2 不兼容，禁止使用 |

根因：推理引擎基于 llama.cpp PR #22836，**只支持 STQ1_0 量化格式**。Hy-MT2 虽然也走 STQ 路径（见 `docs/internal/hy_mt2_1_8b_model_notes.md:11`），但其 STQ 变体不被当前引擎正确反量化，导致输出全是乱码 token。非 STQ 的标准量化文件则直接加载失败。

### 下载源：ModelScope

- 模型下载 URL **必须使用 ModelScope**（`modelscope.cn`），国内速度快。禁止使用 `hf-mirror.com` 或其他 HuggingFace 镜像。
- ModelScope resolve URL 格式：`https://modelscope.cn/models/{namespace}/{model}/resolve/master/{filename}`
- **命名空间差异**：Hy-MT2 在 HuggingFace 上是 `tencent/`，在 ModelScope 上是 `AngelSlim/`。修改 URL 时必须确认 ModelScope 上的实际命名空间。
- 下载 URL 出现在 4 处，必须同步修改：`supported_model_info.dart`、`TranslatorChannelHandler.kt`（Android）、`TranslatorChannelHandler.swift`（macOS）、`model_selection_state_test.dart`（测试）。

## 跨平台原生引擎

`translator_engine.hpp` C++ API 在 macOS 与 Android 间保持不变，平台差异全部隔离在桥接层（`translator_bridge.mm` / `translator_jni.cpp`）。修改推理逻辑时只动共享 header/cpp；修改平台特定行为时只动对应 bridge。

## 编码风格与命名约定

Dart 代码遵循标准 Flutter 格式，两个空格缩进。

- **文件命名** — `snake_case.dart`（源文件与测试文件统一）。
- **类命名** — PascalCase。方法、字段、channel 名称使用 lowerCamelCase。
- **测试文件** — `_test.dart` 后缀，与 `lib/features/` 对应放在 `test/features/` 下。
- **Import** — 项目内部引用使用 `package:ai_offline_translator/`。
- **原生桥接文件** — 按职责命名，例如 `translator_engine.cpp`、`TranslatorChannelHandler.kt`。

Markdown 标题应清晰描述内容，仓库内文件链接使用相对路径。

## UI 与视觉规范

涉及 Flutter UI、页面布局、组件样式、颜色、字体、间距或交互状态的改动，必须先阅读并遵循 `DESIGN.md`。

- `DESIGN.md` 是当前项目的视觉规范来源，基于 MiniMax 风格。
- 首版 UI 应保持工具型翻译界面，不做营销页或 landing page。
- 使用 `DESIGN.md` 中的颜色、圆角、间距、按钮、输入框和卡片规则作为默认设计系统。
- 如 Flutter 默认字体或平台字体无法直接使用 `DM Sans`，应先保持排版比例和层级一致，再单独讨论字体引入。

## 测试规范

新增功能应同步补充测试。Flutter 测试放在 `flutter_app/test/`，文件名使用 `_test.dart` 后缀。原生推理代码在接入 UI 前，应提供小型、可重复的测试或脚本，覆盖模型加载、取消推理和错误处理。

当前仓库的本地推理验证统一使用：

- `uv run scripts/safe_llama_smoketest.py`
- 或 `./scripts/safe_llama_smoketest.ps1`

不要把交互式终端会话当成自动化验证方式。

## 提交与 Pull Request 规范

当前提交历史使用简短祈使句，例如 `Add Flutter gitignore`。继续保持这种风格：简洁、现在时、每次提交聚焦一个变更。

Pull Request 应包含简要说明、变更路径、已执行的验证，以及模型或运行时假设。涉及 UI 的 PR 还应附截图或录屏。

## 安全与配置提示

不要提交 `.env`、本地模型二进制文件、构建产物或平台生成文件。除非仓库明确采用大文件存储策略，否则大型 GGUF 模型应保留在 Git 之外。

## 移动端优先路线

本项目最终目标是移动端离线翻译 App。所有推理、模型管理和构建方案都应优先服务 Android/iOS 可落地性，而不是只服务 macOS 桌面验证。

- macOS 只是首个验证平台，用于快速验证 Flutter UI、模型加载、prompt、取消和错误处理。
- 长期技术路线必须收敛到"Flutter UI + shared native `translator_engine` + thin platform bridge"。
- CLI `llama-completion` 只能作为 macOS 开发期 fallback / smoke test，不应作为移动端架构目标。
- 原生引擎实现应参考 `models/AngelSlim/Hy-MT-demo-apk-technical-report.md`：使用 `llama.cpp` common 层的 chat template、sampler、tokenize/token-to-piece 能力，而不是手写不兼容的 prompt/token 序列。
- 模型应下载或导入到 app 私有目录（Android `files/models/`，iOS/macOS `Application Support/models/`），避免依赖开发机仓库路径。

## 发布与打 Tag 纪律

**铁律：未实际验证通过，不打 tag、不 push release commit。**

- "验证通过"意味着在目标平台上真实运行成功：macOS 要 `flutter run -d macos` 跑通翻译，Android 要 `flutter run -d android` 跑通翻译。
- `flutter analyze` + `flutter test` + `flutter build` 通过 ≠ 功能可用。构建成功只证明编译没报错，不证明运行时正确。
- 缺少目标平台运行环境（如 NDK、设备、模拟器）时，不要打 tag。可以先提交代码，等环境就绪验证通过后再 release。
- 每次打 tag 前，自问："我刚才在目标设备上看到功能正常了吗？"如果答案是否，不要打 tag。
- 发版前确认文档已同步：CHANGELOG 有对应版本条目、README 版本号已更新、官网 `docs/index.html` 功能描述与当前版本一致。

### 2026-05-16 教训

在 Android 端还没有 NDK、没有设备、没有实际编译运行的情况下打了 `v0.0.3` tag 并 push。这违反了"验证后再发布"原则。已回退。

根因：
1. 把"代码写完 + analyze/test/build 通过"等同于"功能完成"。
2. 没有在目标平台实际运行就执行了 release 流程。
3. 被连续输出的节奏带跑，没有停下来做发布前检查。

修正措施：
- release 流程必须包含"在目标平台实际运行验证"步骤。
- 如果环境不满足，可以提交代码，但停留在"开发中"状态，不打 tag。

### v0.0.3 开发过程中的教训

1. **`showModalBottomSheet` 传入 state 快照不会随父组件更新**。Bottom sheet 有独立的 widget 树，父组件 `setState` 不会触发 sheet 重建。必须传入 `ChangeNotifier` 引用并在 sheet 内 `addListener`，才能实现实时状态同步。

2. **`FilledButton.onPressed = null` 会强制使用 disabled 样式**。即使手动设置 `backgroundColor`，`onPressed: null` 也会覆盖为系统 disabled 色。需要保持活跃外观但禁止点击时，用 `AbsorbPointer` + `onPressed: () {}` 替代。

3. **macOS `NSOpenPanel` 必须在主线程调用**。Flutter MethodChannel 回调不一定在主线程，`NSOpenPanel.begin()` 在非主线程调用可能导致文件选择器不弹出或行为异常。

4. **macOS `allowedFileTypes` 在 11.0 后废弃**。应使用 `allowedContentTypes` + `UTType`，但需 `if #available(macOS 11.0, *)` 兼容 10.15 deployment target。

5. **UI 改动后确认用 hot reload 而非重新 build**。hot reload（按 `r`）即可刷新 UI，不需要 `flutter build` 或完全重启。如果 hot reload 无变化，再尝试 hot restart（按 `R`）。

6. **布局从 `CustomScrollView` + slivers 切换到 `Column` + `Expanded` 时**，`TextField` 需要 `expands: true` + `minLines/maxLines: null` 才能撑满父容器，否则还是固定行高。

7. **Android 沉浸式需要三层配合**：`SystemChrome.setEnabledSystemUIMode(SystemUiMode.edgeToEdge)`（Flutter）+ `styles.xml` 透明状态栏/导航栏（Android 原生）+ 自行处理 `MediaQuery.padding`。只做一层不够。

8. **Android 软键盘不要用 `adjustResize`**。翻译工具型 App 键盘拉起时页面不应被压缩，用 `adjustNothing`（AndroidManifest）+ `resizeToAvoidBottomInset: false`（Scaffold）。

9. **Android 必须声明 `INTERNET` 权限**。Flutter 默认不自动添加，网络请求（下载模型）会静默失败。

10. **Dart 字符串插值 `\$` 在 Python 脚本中会被转义**。用 Python 生成 Dart 代码时，`$variable` 会被 Python 解析。应该用硬编码字符串或 `\\$` 转义，避免运行时显示原始 `$value`。

11. **GitHub Release 文件名必须用英文**。中文文件名会被 GitHub 替换为 `.`，导致下载链接不可读。统一用 `AI-Offline-Translator-v<版本号>-<平台>.<ext>`。

12. **Flutter assets 路径不支持 `../` 上级目录引用**。`pubspec.yaml` 的 assets 必须在 `flutter_app/` 目录内，否则图片加载不出来。

13. **`copyWith(clearError: true, errorMessage: msg)` 会清掉显式 errorMessage**。`clearError` 优先级被误设为高于 `errorMessage` 参数，导致同时传两者时 errorMessage 变 null。修正：`errorMessage` 显式传入时优先于 `clearError`。

14. **打字机流式渲染速度用 30-50ms/token**。400ms/字太慢不自然，30-50ms 刚好给人「打字感」又不拖沓。该延迟是 Flutter 渲染层逐字 throttle，不改变原生推理速度。

15. **Release 下载链接优先动态生成**。发布页、README 或文档里如果需要指向最新安装包，先评估能否用 GitHub latest release API / latest download URL / 前端运行时脚本动态生成。只有在确实需要固定版本归档时才硬编码 `v<版本号>` 和资产文件名，避免每次发版后手动同步多个链接。

### 2026-07-30 教训：文档与代码脱节

v0.1.2 发版后发现多处文档与实际状态不一致，导致用户 zero-hugo 报 issue #1（翻译乱码）——这其实是已修复的问题，但文档没同步造成信息矛盾。

具体问题：
1. **README.md 中 Hy-MT2 描述错误** — 写着"已通过翻译验证"，实际实测全乱码。
2. **CHANGELOG 缺 v0.1.2 条目** — 发版时只创建了 GitHub Release，没有同步更新 CHANGELOG.md。
3. **README 版本号停在 v0.1.0** — 发版到 v0.1.2 后未更新。
4. **官网内容过时** — 未提及 ModelScope 下载源和蒙古语竖排功能，Android 权限说明不准确。

根因：只改了代码和 GitHub Release，没有把信息同步到仓库内的所有文档。

修正措施：
- **模型/功能变更后全局搜索旧关键词确认无遗漏**（如 `grep -r "hf-mirror"`、旧模型名、旧版本号）。
- **发版前执行文档检查清单**：CHANGELOG 条目 → README 版本号 → 官网功能描述。
- **修复用户报告的 bug 后及时关闭 issue 并告知修复版本**，不要让已修复的问题继续困扰用户。
- 信息分布在多处（`docs/index.html`、`README.md`、`CHANGELOG.md`、`AGENTS.md`），变更时必须全部同步。

## iOS 分发现实

不上架 App Store 的情况下，iOS 用户只能通过 Xcode 侧载安装 .ipa，每 7 天需重新签名。体验远不如 Android。本项目聚焦 macOS + Android，iOS 暂不投入。

## 发布流程

### 构建发布包

1. 确认两个平台都已实际运行验证通过
2. 构建产物不入 Git，只上传到 GitHub Release

**Android APK：**

```bash
cd flutter_app
flutter build apk --target-platform android-arm64 --release
# 产物：flutter_app/build/app/outputs/flutter-apk/app-release.apk
```

**macOS DMG：**

```bash
# 1. 构建 .app
cd flutter_app
flutter build macos --release
# 产物：flutter_app/build/macos/Build/Products/Release/ai_offline_translator.app

# 2. 打包 DMG（重命名 app、设置 Finder 窗口大小和图标布局）
rm -rf /tmp/dmg-temp; mkdir -p /tmp/dmg-temp
cp -R flutter_app/build/macos/Build/Products/Release/ai_offline_translator.app "/tmp/dmg-temp/AI离线翻译.app"
ln -sf /Applications /tmp/dmg-temp/Applications
hdiutil create -volname "AI-Offline-Translator" -srcfolder /tmp/dmg-temp -ov -format UDRW -fs HFS+ /tmp/ai-translator-raw.dmg
hdiutil attach /tmp/ai-translator-raw.dmg -readwrite -noverify -noautoopen -quiet
osascript -e 'tell application "Finder"
  tell disk "AI-Offline-Translator"
    open
    set current view of container window to icon view
    set toolbar visible of container window to false
    set statusbar visible of container window to false
    set the bounds of container window to {260, 180, 740, 500}
    set viewOptions to the icon view options of container window
    set arrangement of viewOptions to not arranged
    set icon size of viewOptions to 80
    set position of item "AI离线翻译.app" of container window to {160, 180}
    set position of item "Applications" of container window to {360, 180}
    close
    open
    update without registering applications
    delay 2
  end tell
end tell'
sync; sleep 2; hdiutil detach "/Volumes/AI-Offline-Translator" -quiet; sleep 2
hdiutil convert /tmp/ai-translator-raw.dmg -format UDZO -o AI-Offline-Translator-v<版本号>-macos.dmg -ov
rm -f /tmp/ai-translator-raw.dmg; rm -rf /tmp/dmg-temp
```

### Release 文件命名规范

**必须使用英文文件名**，GitHub Release 会吞掉中文字符。统一格式：

```text
AI-Offline-Translator-v<版本号>-android-arm64.apk
AI-Offline-Translator-v<版本号>-macos.dmg
```

示例：

```text
AI-Offline-Translator-v0.0.3-android-arm64.apk
AI-Offline-Translator-v0.0.3-macos.dmg
```

### 创建 GitHub Release

```bash
# 打 tag
git tag -a v<版本号> -m "v<版本号>: 简要描述"
git push origin v<版本号>

# 创建 Release 并上传安装包
gh release create v<版本号> \
  "AI-Offline-Translator-v<版本号>-android-arm64.apk" \
  "AI-Offline-Translator-v<版本号>-macos.dmg" \
  --title "v<版本号> - 标题" \
  --notes "更新内容说明"
```

### macOS 分发说明

- 未签名的 `.dmg`，用户首次打开需要右键 → 打开（绕过 Gatekeeper）
- 不上架 App Store，通过 GitHub Releases 分发

### Android 分发说明

- 自签名 debug APK，不上架 Google Play
- 仅支持 `arm64-v8a`

## Android ABI 约束

- **禁止 x86 / x86_64 ABI**。只支持 `arm64-v8a`。
- 模拟器也必须使用 ARM64 系统镜像（如 `system-images;android-34;google_apis;arm64-v8a`），不允许使用 x86_64 镜像。
- 如果需要模拟器验证，优先使用 Android Studio 创建 Apple Silicon 原生 ARM64 AVD，或使用真机。

---
> Source: [kelegele/ai-offline-translator](https://github.com/kelegele/ai-offline-translator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
