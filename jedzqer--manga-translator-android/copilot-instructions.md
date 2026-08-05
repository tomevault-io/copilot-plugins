## manga-translator-android

> - 重点提供开发环境、构建方式、模块路径和常见改动入口。

# Manga Translator 开发文档

## 文档目的
- 面向 AI AGENT 。
- 重点提供开发环境、构建方式、模块路径和常见改动入口。

---

## 快速导航

| 场景 | 关键位置 |
|-----|---------|
| 翻译主流程 | `app/src/main/java/com/manga/translate/translation/TranslationPipeline.kt` |
| 多供应商调度 | `SettingsFragment.kt`、`SettingsStore.kt`、`ProviderProfileStore.kt`、`TranslationProviderScheduler.kt`、`FolderTranslationCoordinator.kt` |
| 页面区域检测 | `app/src/main/java/com/manga/translate/detection/PageRegionDetector.kt`、`BubbleDetector.kt`（Manga109 气泡分割）、`TextDetector.kt`（yolo11n-text 游离文字） |
| OCR 相关 | `app/src/main/java/com/manga/translate/ocr/OcrSharedTools.kt`、`OcrEngine.kt`、`MangaOcrMobile.kt`、`KoreanOcr.kt`、`model/OcrApiFormat.kt`、`network/BaiduAccessTokenManager.kt`、`network/LlmClient.kt` |
| 漫画库 / 导入导出 | `LibraryFragment.kt`、`LibraryRepository.kt`、`LibraryImportExportCoordinator.kt` |
| 阅读与气泡编辑 | `ReadingFragment.kt`、`ReadingSessionViewModel.kt`、`ReadingImageTransformController.kt`、`FloatingTranslationView.kt`、`BubbleRenderer.kt`、`BubbleTextScaling.kt` |
| 设置页与参数持久化 | `SettingsFragment.kt`、`SettingsStore.kt`、`ApiSettingsStore.kt`、`OcrSettingsStore.kt`、`RenderSettingsStore.kt`、`LlmParameterStore.kt`、`ProviderProfileStore.kt` |
| 后台翻译保活 / 恢复 | `TranslationKeepAliveService.kt`、`TranslationTaskPersistence.kt`、`LibraryUiBridge.kt`、`ServiceLibraryUiCallbacks.kt` |
| 悬浮窗翻译 | `FloatingBallOverlayService.kt`、`FloatingDetectionOverlayView.kt`、`FloatingBubbleTranslationCoordinator.kt`、`FloatingEmptyBubbleCoordinator.kt` |
| 更新检测 | `UpdateChecker.kt`、`update.json` |
| 应用入口 / 共享依赖 | `MangaTranslateApp.kt`、`di/AppContainer.kt`、`MainActivity.kt` |

---

## 开发环境与构建

### 推荐环境
- 操作系统：Windows + WSL2 (Ubuntu) 或原生 Linux。
- JDK：17。
- Kotlin：2.0.0+。
- Gradle：使用项目自带 Wrapper。
- Android SDK：
  - `platforms;android-36`
  - `build-tools;36.0.0`
  - `platform-tools`

### 环境变量
```bash
export ANDROID_HOME=/home/jed/Android
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

### 常用命令
```bash
./gradlew tasks
./gradlew :app:assembleDebug
./gradlew :app:assembleRelease
./gradlew :app:compileDebugKotlin
./gradlew :app:lint
```

### 命令超时建议
- Gradle 冷启动、Kotlin 编译和定向单元测试可能超过默认的 120 秒；执行 `:app:compileDebugKotlin` 或 `:app:testDebugUnitTest` 时，工具超时至少设置为 `300000ms`（5 分钟）。
- 执行 `:app:assembleDebug`、`:app:assembleRelease`、完整单元测试和 `:app:lint` 时，工具超时至少设置为 `600000ms`（10 分钟）。

### 构建产物
- Debug APK：`app/build/outputs/apk/debug/`
- Release APK：`app/build/outputs/apk/release/`
- Lint 报告：`app/build/reports/`

### 仓库与依赖说明
- 单模块 Android 项目。
- 仓库源统一在 `settings.gradle.kts`。

---

## 仓库结构

```text
.
├── app/
├── assets/
├── dev_doc/
├── gradle/
├── build.gradle.kts
├── settings.gradle.kts
├── gradle.properties
├── gradlew
├── update.json
└── README.md
```

### 关键目录
- `app/`：Android 应用源码与资源。
- `assets/`：模型、Prompt 等静态资源，构建时并入 app assets。
- `dev_doc/`：开发计划与开发文档。

---

## 模块路径定位

源码根目录为 `app/src/main/java/com/manga/translate`，按领域划分为 `app`、`background`、`library`、`reader`、`floating`、`settings`（设置 UI 位于 `settings/ui`）、`translation`、`network`、`ocr`、`detection`、`rendering`、`storage`、`model`、`platform` 和 `di`。

详细职责与依赖方向见 `dev_doc/package_architecture.md`。

### 入口层
- `MangaTranslateApp.kt`：Application 入口，初始化主题、语言、日志和全局依赖容器。
- `di/AppContainer.kt`：共享依赖统一创建入口。
- `MainActivity.kt`：主 Activity，负责三页导航、全局状态展示、更新检查等。
- `MainPagerAdapter.kt`：三标签页索引维护。

### UI 层
- `LibraryFragment.kt`：漫画库主页面与文件夹详情入口。
- `ReadingFragment.kt`：阅读页面容器。
- `SettingsFragment.kt`：设置页主入口。
- `SettingsStore.kt`：设置门面，向外保持统一 API，向内委托给多个子 store。
- `ApiSettingsStore.kt`、`OcrSettingsStore.kt`、`RenderSettingsStore.kt`、`AppSettingsStore.kt`、`LlmParameterStore.kt`、`ProviderProfileStore.kt`：按领域拆分后的设置持久化实现。

### Library 相关
- `LibraryRepository.kt`：漫画库目录与文件操作的核心数据入口。
- `LibraryImportExportCoordinator.kt`：导入导出流程协调。
- `FolderTranslationCoordinator.kt`：文件夹和批量翻译协调。
- `LibrarySelectionController.kt`：列表选择态控制。
- `LibraryDialogs.kt`：通用对话框封装。

### Reading 相关
- `ReadingSessionViewModel.kt`：阅读会话共享。
- `ReadingImageTransformController.kt`：阅读页手势与矩阵控制。
- `ReadingBitmapDecoder.kt` / `ReadingRegionImageView.kt`：阅读解码。普通页优先 `ARGB_8888`、detail≈屏×3；长图/超高分图走区域分块，布局坐标用源分辨率，`decodeSample` 随缩放动态降到 1 以在放大时接近原画，低分 tile 可作 fallback。
- `ReadingEmptyBubbleCoordinator.kt`：阅读页空白气泡补翻，OCR 后批量提交空白气泡翻译。
- `WebtoonReadingAdapter.kt`：条漫模式列表适配。
- `BubbleRenderer.kt`：气泡渲染。
- `FloatingTranslationView.kt`：阅读页 / 条漫页翻译覆盖层。
- `BubbleShapePaths.kt`：普通气泡框轮廓 path 组装与回缩处理。
- `BubbleTextScaling.kt`：文本缩放共享算法，提供布局拟合判定、密度自适应文字区域解析（按字数与气泡面积计算密度，不足时自动扩大路径）、水平文本二分搜索与路径缩放，三个渲染端点 (`BubbleRenderer`、`FloatingTranslationView`、`FloatingDetectionOverlayView`) 均调用该模块。

### 翻译与 OCR
- `TranslationPipeline.kt`：翻译主流程编排。
- `TranslationProviderScheduler.kt`：多供应商调度相关类型，包含附加供应商配置结构、加权候选项、页面级供应商上下文和调度器。
- `PageRegionDetector.kt`：页面区域检测公共模块；先用 Manga109 YOLO11n-seg 模型检测普通气泡并屏蔽对应区域，再用 yolo11n-text 补检游离文字，最后做 overlap/filter 与 `source` 组装。
- `LlmClient.kt`：LLM 请求客户端；文本气泡翻译已支持结构化 `items[{id,text}] -> items[{id,translation}]` 协议解析，当前网络层基于 `OkHttp`。主 AI 请求默认会读取设置页里的"API 最大重试次数 (1–50，默认 3)"并在可重试错误时按固定延时自动重试。OCR API 请求支持根据 `OcrApiFormat` 分发到 OpenAI 兼容端点或百度 AI OCR 端点，百度 AI 模式由 `BaiduAccessTokenManager` 管理 OAuth 令牌。
- OpenAI 兼容接口对 `API 地址` 采用统一补全策略：填写的地址若已以 `/chat/completions` 结尾则原样使用，否则自动追加 `/chat/completions`；模型列表地址同理追加 `/models`。不再自动补全 `/v1`，也不再为智谱维护独立分支。接入智谱时设置页 `API 格式` 选择 `OpenAI 兼容`，`API 地址` 直接填 `https://open.bigmodel.cn/api/paas/v4`（Coding 场景填 `https://open.bigmodel.cn/api/coding/paas/v4`），火山引擎等带 `/api/v3` 前缀的地址同理，鉴权仍使用 `Bearer API Key`。
- OpenAI Responses 格式（`ApiFormat.OPENAI_RESPONSES`）走 `/responses`：地址若已以 `/responses` 结尾则原样使用，否则自动追加 `/responses`；模型列表仍用 `/models`。请求体使用 `model` + `input`（消息数组）+ 可选 `instructions`（system prompt），采样参数为 `temperature` / `top_p` / `max_output_tokens`；响应优先读 `output_text`，否则从 `output[].content[].text`（`output_text`/`text` 类型）拼接。文本翻译与 VL 图片翻译均支持该格式；OCR API 仍仅支持 OpenAI 兼容 chat 与百度 AI。
- `TextBubbleTranslationCoordinator.kt`：共享文本气泡翻译入口，统一结构化请求、LLM 调用、按 `id` 回填、缺失项留空/多余项丢弃，以及 glossary 回传。
- `FloatingBubbleTranslationCoordinator.kt`：悬浮窗气泡翻译协调，负责悬浮窗特有的缓存与回退策略。
- `OcrSharedTools.kt`：OCR 共享工具模块，集中放 OCR 引擎注册、区域识别、裁剪、行识别和 OCR 文本归一化辅助。
- `OcrEngine.kt`：OCR 抽象接口。
- `MangaOcr.kt`、`MangaOcrMobile.kt`、`EnglishOcr.kt`、`KoreanOcr.kt`：当前启用的本地 OCR 实现。日文支持 `MangaOcr` 与 `MangaOcr Mobile`，默认走后者；英文/拉丁语系以及简体中文、繁体中文、中英混合当前统一走 `PP-OCRv6_small_rec`；`PP-OCR` 已移除。
- `EnglishLineDetector.kt`：英/韩行检测模型封装。

### 检测与模型支持
- `PageRegionDetector.kt`：页面区域检测编排层，对上提供统一 `PageRegion` 列表；主翻译与悬浮窗均调用该模块，内部串联普通气泡检测与游离文字补检。
- `BubbleDetector.kt`：`models/detection/manga109-segmentation-bubble.onnx`，YOLO11n-seg 固定 1600×1600 输入，单类 `balloon` 输出检测框与 400×400 原型掩码，并提取 `maskContour` 供气泡渲染；游离文字仍由 `TextDetector.kt` 单独补检。
- `TextDetector.kt`：`models/detection/yolo11n-text.onnx` 游离文字补检；推理前会把普通气泡区域扩张后涂白，固定使用 40% 置信度阈值。
- `OnnxRuntimeSupport.kt`：ONNX Runtime 会话与线程相关支持。
- `VerticalTextSymbolConverter.kt`：竖排文本辅助处理。

### 悬浮窗翻译
- `FloatingBallOverlayService.kt`：悬浮窗服务主入口；区域检测复用 `PageRegionDetector`（Manga109 气泡分割 + yolo11n-text），气泡带 `BubbleSource` 和可选 `maskContour`。
- `FloatingDetectionOverlayView.kt`：悬浮结果绘制与编辑；有 `maskContour` 的 balloon 走轮廓 path，游离/手动框仍按悬浮窗形状设置绘制。
- `FloatingEmptyBubbleCoordinator.kt`：悬浮编辑态补翻。
- `ProjectionCaptureSession.kt`：抓屏会话封装。
- `FloatingTranslationView.kt`：阅读页 / 条漫页翻译覆盖层（非悬浮窗 overlay）。

当前气泡框绘制已拆成两条独立分支：
- 普通气泡框：用于阅读页、条漫页和导出图片，设置入口在 `SettingsFragment.kt` 的“普通气泡框设置”，渲染主入口是 `FloatingTranslationView.kt`、`BubbleRenderer.kt`、`BubbleShapePaths.kt`。
- 悬浮窗气泡框：仅用于悬浮窗 overlay，设置入口在 `SettingsFragment.kt` 的”悬浮窗气泡设置”，渲染主入口是 `FloatingDetectionOverlayView.kt`；拥有独立的最小字号设置；检测侧与主流程共用 Manga109 气泡分割和 yolo11n-text 游离文字区分。
- 字体设置：普通气泡框与悬浮窗气泡框共用一套全局字体设置，设置入口在 `SettingsFragment.kt` 的“字体设置”；当前保留系统默认、已上传字体列表（可切换/删除）与字体加粗，上传的字体会持久保存在私有 `custom_fonts` 目录。

### 数据与状态存储
- `TranslationStore.kt`：翻译结果读写。
- `OcrStore.kt`：OCR 缓存读写。
- `GlossaryStore.kt`：译名表读写。
- `ExtractStateStore.kt`：译名抽取状态。
- `FloatingTranslationCacheStore.kt`：悬浮窗翻译缓存，持久化到 APP `cacheDir`，清除应用缓存即可清空。
- `TranslationTaskPersistence.kt`：后台翻译任务描述持久化，供前台服务恢复未完成任务。
- `SettingsStore.kt`：全局设置门面；主调用入口保持不变。
- `ApiSettingsStore.kt`、`OcrSettingsStore.kt`、`RenderSettingsStore.kt`、`AppSettingsStore.kt`、`LlmParameterStore.kt`、`ProviderProfileStore.kt`：设置读写的分层实现。
- `ReadingProgressStore.kt`：阅读进度。
- `CrashStateStore.kt`：崩溃状态。
- `UpdateIgnoreStore.kt`：忽略更新版本。

### 基础设施与配置
- `AppLogger.kt`：日志。
- `TranslationKeepAliveService.kt`：翻译前台服务保活；当前也是漫画库文件夹/批量翻译的实际任务宿主。
- `LibraryUiBridge.kt`：Library 页 UI 回调桥，负责把 Service 侧状态转发给当前可见的界面。
- `ServiceLibraryUiCallbacks.kt`：提供给 `TranslationKeepAliveService` / `FolderTranslationCoordinator` 使用的 UI 代理实现。
- `UpdateChecker.kt`：更新检测。
- `PromptAssetResolver.kt`：Prompt 资源解析。
- `ErrorDialogFormatter.kt`：错误文案格式化。

---

## 资源与配置定位

### 资源目录
- `app/src/main/res/`：布局、drawable、strings、theme 等 Android 资源。
- `app/src/main/assets/`：应用内静态资源。
- 根目录 `assets/`：模型与 Prompt 资源。

### assets 合并配置
```kotlin
sourceSets["main"].assets.srcDirs("src/main/assets", "../assets")
```

### 常用资源入口
- `app/src/main/AndroidManifest.xml`：权限、Activity、Service、FileProvider。
- `app/src/main/res/layout/`：主要页面与对话框布局。
- `app/src/main/res/values/strings.xml`：简体中文文案。
- `app/src/main/res/values-b+zh+Hant/strings.xml`：繁体覆盖文案。
- `app/src/main/res/values/themes.xml`：主题定义。
- `app/src/main/res/xml/`：`FileProvider`、语言配置等 XML。

### Prompt 文件
- `assets/prompts/llm_prompts.json`
- `assets/prompts/llm_prompts_abstract.json`
- `assets/prompts/llm_prompts_FullTrans.json`
- `assets/prompts/float_llm_prompts.json`
- `assets/prompts/vl_bubble_prompts.json`
- `assets/prompts/ocr_prompts.json`

繁体界面优先读取同名 `_hant` 文件，不存在时回退基础文件。

---

## 核心业务概览

### 漫画库
- 入口：`LibraryFragment.kt`
- 核心数据：`LibraryRepository.kt`
- 导入导出：`LibraryImportExportCoordinator.kt`
- 批量翻译：`LibraryFragment.kt` 发起任务，`TranslationKeepAliveService.kt` 托管后台执行，`FolderTranslationCoordinator.kt` 负责编排具体翻译流程。
- 首页普通文件夹和文件夹集合右侧统一提供三点菜单，包含重命名、删除和编辑标签；长按仍进入批量选择，但批量栏不再提供重命名。
- 首页「漫画项目」标题右侧提供排序控件：左侧文案在「名称 / 时间」间切换排序字段，右侧箭头切换正序/倒序；默认按添加时间降序（新的在上）。排序偏好经 `LibraryPreferencesGateway.kt` 持久化（`library_sort_field`、`library_sort_ascending`），列表排序在 `LibraryRepository.listFolders` / `sortFolders`。
- 用户标签通过 `LibraryPreferencesGateway.kt` 按文件夹路径持久化，显示样式复用 `bg_status_chip`。点击内置状态标签或用户标签会筛选首页列表，再次点击同一标签或点击当前筛选标签可清除筛选。

当前库页支持普通文件夹和文件夹集合两类目录，集合用于组织子章节，不直接存放图片。

### 阅读
- 入口：`ReadingFragment.kt`
- 状态共享：`ReadingSessionViewModel.kt`
- 模式支持：横向阅读、条漫滚动。
- 普通文件夹首次上传图片、压缩包/PDF 首次导入、以及合集首次导入章节时，会按图片长宽比自动识别是否更接近条漫；命中后自动把该作品的阅读方式切到 `WEBTOON_SCROLL`，否则保持普通横向阅读。
- 气泡渲染与编辑改动通常从 `BubbleRenderer.kt`、`ReadingImageTransformController.kt`、`WebtoonReadingAdapter.kt` 入手。

### 翻译
- 主入口：`TranslationPipeline.kt`
- 大致流程：
  1. 检查 OCR 缓存
  2. 调用 `PageRegionDetector.kt` 做页面区域检测
  3. OCR 或 VL 直翻
  4. LLM 翻译
  5. 保存 `*.ocr.json` / `*.json`

全文速译、普通逐页翻译、文件夹批量翻译都在现有协调器基础上展开，不建议绕开 `TranslationPipeline.kt` 直接拼流程。

当前漫画库后台翻译链路已调整为：
- `LibraryFragment.kt` 不再持有翻译协程生命周期，而是将任务描述交给 `TranslationKeepAliveService.startTranslationTask(...)`。
- `TranslationKeepAliveService.kt` 持有自己的 `CoroutineScope` 和 `Job`，真正负责文件夹翻译、合集翻译和批量翻译的执行生命周期。
- `TranslationKeepAliveService.kt` 在后台翻译结束后会补发系统通知（成功 / 失败 / 取消），点击后回到漫画库页查看结果。
- `FolderTranslationCoordinator.kt` 仍负责翻译编排，但不再负责启动/停止保活服务；它现在向上返回真实任务 `Job`，由 Service 持有。
- `TranslationTaskPersistence.kt` 会在任务运行期间写入任务描述（便于同进程内状态/崩溃时清理）；**进程重启后一律丢弃，不再自动恢复**。页级进度依赖 `*.ocr.json` / `*.json` 缓存，用户再次点翻译时会从文件续跑。冷启动与 Service 被系统拉起时都会 `clear()`；未捕获异常时也会立刻清空。
- `LibraryUiBridge.kt` 与 `ServiceLibraryUiCallbacks.kt` 用来把后台 Service 中的状态、Toast、刷新请求和模型错误对话框转发给当前附着的 Library UI。

当前文本气泡翻译逻辑已做一轮收敛：
- `TextBubbleTranslationCoordinator.kt` 是文本气泡翻译共享入口。
- `TranslationPipeline.kt` 的普通翻译与全文翻译、`ReadingEmptyBubbleCoordinator.kt` 的阅读页空白气泡补翻、`FloatingBubbleTranslationCoordinator.kt` 的悬浮窗文本气泡翻译都复用这一入口。
- 当前文本气泡翻译不再依赖 `<b>...</b>` 标签数量匹配，而是使用结构化 JSON 协议按 `id` 对齐；响应项只接受 `translation`、`translated_text` 或 `translatedText` 译文字段，不把输入侧的 `text` 字段当作译文。
- 当模型返回重复、额外或缺失的气泡 `id` 时，按模型响应错误处理：先执行静默重试，仍失败则进入现有模型错误弹窗。译文与对应 OCR 原文相同是合法结果，不做原文回显校验。模型必须为无意义气泡保留 `id` 并返回空译文；软件收到完整 `id` 对应的空译文后会移除该气泡。该校验只约束新模型响应，不迁移或重新判定已有翻译结果。
- 如果后续要调整文本气泡翻译的结构化 JSON 协议、按 `id` 对齐策略、glossary 回传或统一错误格式，优先修改 `TextBubbleTranslationCoordinator.kt` 和 `LlmClient.kt`。

当前主文本翻译链路已接入多供应商调度：
- 设置入口位于 `SettingsFragment.kt` 主模型配置区的“多供应商调度”按钮，对话框布局在 `dialog_multi_provider_scheduling.xml` 与 `item_additional_translation_provider.xml`。
- 配置持久化统一通过 `SettingsStore.kt` 门面进入，供应商相关序列化当前由 `ProviderProfileStore.kt` 负责；附加供应商字段固定为 `name`、`apiUrl`、`apiKey`、`modelName`、`weight`、`enabled`。
- 主供应商始终参与主文本翻译调度，权重固定为 `10`；附加供应商只要 `enabled` 且配置完整，就会被加入主文本供应商池。
- 当前调度范围只覆盖主文本翻译任务，也就是普通逐页翻译、全文速译、合集/章节标准翻译和相关重译路径；不接入 OCR、悬浮窗专用翻译设置、VL 图片翻译。
- `TranslationPipeline.kt` 仍保持单次请求只面向一个 `ApiSettings`；页面级供应商选择、失败切换和并发调度位于 `FolderTranslationCoordinator.kt` 外围执行层。
- 页面任务开始时会选定本页的首选供应商；若本页请求失败，再在该页尚未尝试过的候选项中继续切换。
- 单页内不会把不同气泡拆给多家供应商；一个页面的一次成功翻译结果只来自一家供应商。
- 当前 glossary 在并发页面任务中采用“读取快照、成功后串行合并”的方案，不保证页面之间实时看到彼此刚新增的 glossary。
- 文件夹级翻译设置现在包含两个和 glossary 相关的开关：`全文速译` 与 `译名处理`。
- 当 `全文速译` 开启时，仍按原流程先 OCR 全部页面、再统一抽取译名、再执行整页翻译；此时 `译名处理` 视为固定开启，界面上禁用单独修改。
- 当 `全文速译` 关闭且 `译名处理` 开启时，普通逐页翻译会继续沿用当前并发方案，并在页面翻译成功后把模型回传的 `glossary_used` 串行合并回 `glossary.json`。
- 当 `全文速译` 关闭且 `译名处理` 关闭时，普通逐页翻译仍会把现有 `glossary.json` 当作上下文发送给模型，但不会提取、合并或写入新的译名；这个模式就是当前推荐的“仅复用已有译名 + 并发翻译”路径。
- `CrossPageBubbleMerger.kt` 只在条漫阅读方式 (`WEBTOON_SCROLL`) 下参与全文速译和普通逐页翻译的预处理；普通横向阅读模式不会做跨页气泡合并。

当前 OCR 相关逻辑已做第一轮收敛：
- `OcrSharedTools.kt` 统一提供 `OcrEngineRegistry` 和 `BubbleTextRecognizer`。
- 阅读页空白气泡补翻、悬浮窗空白气泡补翻、悬浮窗即时识别、`TranslationPipeline` 中的 OCR 识别入口都优先走共享模块。
- 如果后续要调整 `JA/EN/KO` 识别策略、本地/API OCR fallback、行检测或裁剪规则，优先从 `OcrSharedTools.kt` 入手，而不是分别改 `Coordinator` 或 `Service`。

当前页面区域检测链路也已独立成公共模块：
- `PageRegionDetector.kt` 先调用 Manga109 YOLO11n-seg：`balloon` → `BubbleSource.BUBBLE_DETECTOR`（同时保留分割轮廓）；随后屏蔽这些普通气泡区域并调用 yolo11n-text：补检结果 → `BubbleSource.TEXT_DETECTOR`（游离气泡），再做 overlap/filter 与合并。
- Manga109 气泡模型固定使用 1600×1600 输入，应用侧按比例 letterbox 并传入 RGB 0–255 值，普通气泡候选使用 15% 的最低置信度下限。普通方图、横图和漫画页不再按分辨率二维切块，而是整页缩放后分别执行一次气泡检测与文字检测；只有高度至少 2048px 且高宽比超过 2.0 的超长竖图才分块。长图气泡与文字 tile 均覆盖整幅宽度，只沿 Y 轴重叠滑动，yolo11n-text 的文字 tile 高度约为页面宽度的 1.5 倍并保持 30% 重叠，检测结果回映射到页面坐标后去重、合并，并启用长图异常大框过滤。
- `TranslationPipeline.kt` 与 `FloatingBallOverlayService.kt` 均调用该模块；主流程保留缓存/OCR/落盘编排，悬浮窗对抓屏 bitmap 直接 `detect(bitmap)`。
- 如果后续要调整去重阈值、`BubbleSource` 或 `maskContour` 的组装逻辑，优先修改 `PageRegionDetector.kt` / `BubbleDetector.kt`。

当前网络请求链路说明：
- `LlmClient.kt` 已从 `HttpURLConnection` 切换为 `OkHttp`。
- 文字翻译、图片翻译、OCR API 请求、模型列表获取都统一走 `OkHttp` 请求执行。
- 主 AI 请求的最大重试次数由 `SettingsStore.kt` 统一持久化，设置入口在 `SettingsFragment.kt` 的主 API 配置区；当前默认值是 `3`，上限是 `50`。
- 当前自动重试主要用于 `NETWORK_ERROR`、`TIMEOUT`、`HTTP 408`、`HTTP 429`、`HTTP 5xx` 以及响应体明确表现为“接口暂时不可用”的场景；默认每次重试前固定等待数秒。
- 如果后续要调整连接超时、读写超时、取消传播、统一请求头或重试策略，优先修改 `LlmClient.kt`，不要分散到各个 `Coordinator`。

### 悬浮窗翻译
- 服务入口：`FloatingBallOverlayService.kt`
- 抓屏：`ProjectionCaptureSession.kt`
- 绘制与编辑：`FloatingDetectionOverlayView.kt`
- 共享翻译能力：`FloatingBubbleTranslationCoordinator.kt`
- 空白气泡补翻编排：`FloatingEmptyBubbleCoordinator.kt`

当前悬浮窗文本翻译链路也沿用相同结果安全约束：
- 新的文本翻译响应不会把 OCR 输入字段误当作译文。
- 悬浮窗补翻会继续尝试 OCR / VL 补齐空白气泡；补完后仍为空的气泡保持留空。

### 错误弹窗
- 通用模型错误弹窗入口：`ModelErrorDialogs.kt`
- 页面内调用封装：`LibraryDialogs.kt`
- 阅读页空白气泡补翻仍由 `ReadingFragment.kt` 在界面层接住 `LlmResponseException` / `LlmRequestException`，统一处理真正的模型格式错误或 API 错误。
- 模型输出气泡 `id` 重复、额外或缺失会作为模型响应错误，经静默重试后仍失败时进入模型错误弹窗；原文回显不再视为错误。
- 悬浮窗翻译仍由 `FloatingBallOverlayService.kt` 自己处理弹窗与 overlay window type，但数量不匹配/缺项留空不再触发模型错误弹窗。

### 设置与供应商配置
- UI：`SettingsFragment.kt`
- 门面：`SettingsStore.kt`
- 领域持久化：`ApiSettingsStore.kt`、`OcrSettingsStore.kt`、`RenderSettingsStore.kt`、`AppSettingsStore.kt`、`LlmParameterStore.kt`、`ProviderProfileStore.kt`
- 命名配置文件：`files/ai_provider_profiles.json`

当前与气泡框渲染直接相关的设置分为两组：

- 普通气泡框设置：普通气泡回缩、普通气泡不透明度、游离气泡回缩、游离气泡不透明度、文字密度下限（SeekBar）、横向/竖向排版；作用范围是阅读页、条漫页、导出图片。
- 文字密度定义为气泡文字区域面积（sp²）除以字符数。排版时会自动求填充气泡的最优字号；若面积密度低于下限，则按比例一次性扩大气泡路径以确保文字不会过于拥挤。
- 这里的”游离气泡”仅指普通模式下由 yolo11n-text 补检产出的框（`BubbleSource.TEXT_DETECTOR`），不包含用户手动新增气泡；手动新增气泡继续按普通气泡参数渲染。
- 悬浮窗气泡设置：大小外扩/内缩、不透明度、矩形/内接椭圆、横向/竖向排版、文字密度下限（SeekBar）；作用范围仅限悬浮窗 overlay，各项参数与普通气泡框完全独立。
- 字体设置：通过独立按钮统一控制普通气泡框与悬浮窗气泡框的字体；当前不再提供系统内置字体列表和网络字体入口，只保留系统默认、已上传字体列表（可切换与删除）与字体加粗；字体文件由 `BubbleFontResolver` 管理。

当前 AI 供应商相关设置分为三层：
- 主供应商设置：主 API 地址、Key、模型、API 格式、超时、重试等，仍由设置页主模型区域直接维护。
- 多供应商调度：只维护附加翻译供应商列表，每项只配置 API 地址、API Key、模型名称、访问权重、启用状态；主供应商权重固定为 `10`，不提供编辑入口。
- 自定义请求参数：每项参数默认作用于主供应商，也可以单独指定到 OCR API 或某个附加供应商；请求发出时会按当前命中的供应商筛选，只附加属于该供应商的参数。
- AI 供应商配置档：保存和应用 profile 时会同时带上主供应商、OCR、悬浮窗翻译覆盖、LLM 参数、按供应商分配的自定义请求参数以及多供应商调度配置。

当前附加供应商有几个重要约束：
- 附加供应商沿用主设置中的 `apiFormat`、API 超时、最大重试次数和 LLM 参数，不提供逐供应商覆盖。
- 自定义请求参数支持按供应商分别配置；同一个键可以分别配置给主供应商、OCR API、不同附加供应商，但同一目标内不允许重复键。
- 禁用项允许保留内容并保存，但运行时不会加入供应商池。
- 保存时全空项会被自动忽略；启用项若地址、Key、模型缺失，或权重不是正整数，则禁止保存。

### 设置弹窗 UI 风格约定
- 设置类弹窗优先向 `dialog_ocr_settings.xml` 对齐视觉节奏，再扩展到 `dialog_normal_bubble_render_settings.xml`、`dialog_floating_bubble_render_settings.xml` 等其他设置弹窗。
- 外层 `ScrollView` 默认开启 `android:clipToPadding="false"`，避免弹窗内容靠边太紧或滚动时贴边。
- 内容根节点默认使用 `20dp` 外边留白，如果没有明确理由，新设置弹窗沿用这一基线。
- 输入项之间维持当前设置页统一节奏：说明文案后首个输入项通常 `12dp` 顶部间距，后续输入项也保持 `12dp` 间距，开关类控件通常使用 `14dp` 顶部间距。

---

## 依赖管理

- 当前未引入完整 DI 框架，使用轻量 `AppContainer`。
- `AppContainer` 生命周期与 `Application` 对齐。
- `Activity` / `Fragment` / `Service` 优先通过 `context.appContainer` 获取共享依赖，不要重复各自创建核心对象。
- 当前 `TranslationKeepAliveService.kt` 也通过 `context.appContainer` 创建 `TranslationPipeline`、`FolderTranslationCoordinator` 等核心依赖。
- OCR 共用依赖已经通过 `AppContainer` 统一暴露，包括 `OcrEngineRegistry` 和 `BubbleTextRecognizer`。
- 新增 OCR 入口时，优先复用共享 OCR 依赖，不要在业务类里重复持有 `MangaOcr`、`EnglishOcr`、`KoreanOcr`、`EnglishLineDetector` 实例。

---

## 数据存储约定

### 漫画库目录
`getExternalFilesDir()/manga_library/`

### 主要文件
| 文件 | 说明 |
|-----|------|
| `*.json` | 翻译结果 |
| `*.ocr.json` | OCR 缓存 |
| `glossary.json` | 译名表 |
| `.extract-state.json` | 译名抽取状态 |
| `cache/floating_translate_cache.json` | 悬浮窗翻译缓存 |
| `files/ai_provider_profiles.json` | AI 供应商配置 |
| `SharedPreferences/additional_translation_providers` | 多供应商调度附加供应商配置 |
| `SharedPreferences/translation_task_persistence` | 后台翻译任务描述 |

### Store 对应关系
| 类 | 职责 |
|---|------|
| `TranslationStore.kt` | 翻译结果读写 |
| `OcrStore.kt` | OCR 缓存读写 |
| `GlossaryStore.kt` | 译名表读写 |
| `ExtractStateStore.kt` | 抽取状态读写 |
| `FloatingTranslationCacheStore.kt` | 悬浮缓存读写，仅供悬浮窗翻译使用 |
| `TranslationTaskPersistence.kt` | 后台翻译任务描述读写 |

### SharedPreferences
- 统一入口：`SettingsStore.kt`（门面）
- 文件夹级设置访问：`LibraryPreferencesGateway.kt`
- 阅读进度：`ReadingProgressStore.kt`

常见键按类别理解即可：
- 主模型配置：`api_format`、`api_url`、`api_key`、`model_name`
- 多供应商调度：`additional_translation_providers`
- 主请求控制：`api_timeout_seconds`、`api_retry_count`、`max_concurrency`
- OCR 配置：`ocr_use_local`、`ocr_api_format`、`ocr_api_url`、`ocr_api_key`、`ocr_secret_key`、`ocr_model_name`
- 悬浮窗配置：`floating_api_url`、`floating_api_key`、`floating_model_name`
- 气泡框渲染配置：`normal_bubble_shrink_percent`、`normal_bubble_min_area_per_char_sp`、`horizontal_text_layout`、`floating_bubble_size_adjust_percent`、`floating_bubble_opacity_percent`、`floating_bubble_shape`、`floating_bubble_horizontal_text`、`floating_bubble_min_area_per_char_sp`
- 全局字体配置：`bubble_font`、`bubble_custom_font_file`、`bubble_font_bold`
- 文件夹级配置：`full_translate_enabled_<folder>`、`glossary_processing_enabled_<folder>`、`translation_language_<folder>`、`vl_direct_translate_enabled_<folder>`、`reading_mode_<folder>`
- 漫画库用户标签：`folder_tags_<folder>`
- 漫画库首页排序：`library_sort_field`（`name` / `time`，默认 `time`）、`library_sort_ascending`（默认 `false`，时间降序）
- 应用级配置：`app_language`、`link_source`、主题、并发、超时等

---

## 更新与发布定位

- 更新元数据：根目录 `update.json`
- 更新逻辑：`UpdateChecker.kt`
- 更新弹窗与入口：`MainActivity.kt`
- 更新日志多语言：`update.json` 顶层与 `history[]` 条目支持
  - `changelog`：简体中文（默认 / 回退）
  - `changelog_hant`：繁体中文
  - `changelog_en`：英语
  - `changelog_ru`：俄语
  解析时按当前 APP 界面语言选择对应字段，缺失则回退 `changelog`。

---

## 日志与排查

- 日志入口：`AppLogger.kt`
- 日志目录优先位于外部私有目录上级的 `log/`，回退到 `files/logs/`
- 崩溃：`Thread` 未捕获异常会 `AppLogger.logFatal`（`fd.sync` 刷盘）并写 `crash_latest.log`；协程未捕获异常由 `TranslationKeepAliveService` 的 `CoroutineExceptionHandler` 同样落盘。Native 崩溃（如 ONNX SIGSEGV）不会进入 Java handler，需用 `adb logcat` / tombstone 排查。
- 构建或运行问题优先检查：
  - Gradle / SDK 版本
  - `AndroidManifest.xml` 权限声明
  - `SettingsStore.kt` 及对应子 store 是否把配置写入预期 key
  - `TranslationPipeline.kt` / `LlmClient.kt` / `TranslationProviderScheduler.kt` / `OcrSharedTools.kt` / `Ocr` 实现链路
  - `TranslationKeepAliveService.kt` / `FolderTranslationCoordinator.kt` / `TranslationTaskPersistence.kt` / `LibraryUiBridge.kt` 后台翻译链路
  - 日志输出和相关 `*.json` / `*.ocr.json`

多供应商调度相关排查建议：
- 先确认设置页按钮文案数量是否和已保存的附加供应商条目数一致；这里统计的是已保存条目，不只统计启用项。
- 再看 `SettingsStore.loadMainTranslationProviderPool()` 返回的候选集，确认主供应商和启用的附加供应商都已进入池子。
- 如果页面没有按预期重新翻译，优先检查 `TranslationStore.kt` 里的 metadata 判定；当前页级翻译缓存不再区分 LLM provider / model / apiFormat，只校验源图指纹、翻译模式、语言、prompt 和 OCR 模式。
- 如果并发翻译时 glossary 表现异常，优先检查 `FolderTranslationCoordinator.kt` 的快照读取与串行合并逻辑。
- 如果用户反馈“关闭全文速译后还是会被译名处理拖慢”或“想保留已有译名但不要新增译名”，先检查对应文件夹的 `glossary_processing_enabled_<folder>` 是否已关闭。

---

## 开发规范

- 所有构建操作统一使用 `./gradlew`。
- 不在根 `build.gradle.kts` 重复声明仓库。
- 新增模块、资源、配置或关键流程时，同步更新本文档。
- 文档以“快速定位”优先，避免写入过细的设备适配、线程调参或单次实现细节。
- 修复检测、OCR、渲染或缓存问题时，不得擅自删除、覆盖或使用户已有的 `*.json` 翻译结果失效、隐藏或突然消失；不得仅为强制重新检测而修改 `TranslationStore` 的 metadata 可用性判定、翻译结果版本或翻译缓存兼容规则。确需迁移或废弃已有翻译结果时，必须先明确说明影响并获得用户同意，同时提供保留与恢复方案。

---
> Source: [jedzqer/manga-translator-android](https://github.com/jedzqer/manga-translator-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
