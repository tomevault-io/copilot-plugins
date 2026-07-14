## smartisanclock-revived

> 锤子时钟 7.1.1 的现代 Android 复刻项目。目标是在保留原版视觉、动画、手势和机械触感的前提下，使用当前稳定 Android 工具链实现可维护、可测试、可在现代设备运行的版本。

# Smartisan Clock Revived

锤子时钟 7.1.1 的现代 Android 复刻项目。目标是在保留原版视觉、动画、手势和机械触感的前提下，使用当前稳定 Android 工具链实现可维护、可测试、可在现代设备运行的版本。

## 开发约束

- 源码全部使用 Kotlin，主包为 `com.smartisan.clock`，目录固定为 `app/src/main/kotlin/com/smartisan/clock/`。
- UI 使用 XML Layout、Android View 和自定义 View；不要引入 Jetpack Compose 或 Material 组件重写页面。
- 视觉、尺寸、状态机、动画和触摸参数必须以坚果 R2 原厂时钟 APK、对应系统框架、资源表及反编译证据为准；现代化集中在生命周期、时间基准、Insets、性能与安全边界。
- 新项目 namespace 为 `com.smartisan.clock`，applicationId 为 `app.smartisanclock.revived`。原版包名 `com.smartisanos.clock` 仅用于逆向对照，不得作为新源码包。
- 这是无历史包袱的新项目，不需要兼容原版数据库、SharedPreferences、签名权限或 Smartisan OS 私有服务。
- 不复制原版 OEM 特权权限、导出组件、云同步和系统覆盖层。新增系统能力时使用现代公开 Android API，并坚持最小权限与默认不导出。
- 新增或升级依赖前先查阅 Android/Kotlin 官方文档，只使用最新稳定版；版本统一维护在 `gradle/libs.versions.toml`。
- 保持竖屏原版画布与现有多 Activity 结构，除非原版证据或明确需求要求调整。

## 项目结构

```text
app/src/main/kotlin/com/smartisan/clock/
├── ClockActivity.kt                  # 四页面宿主、生命周期与页面切换
├── ClockApplication.kt               # 闹钟/计时提醒依赖图与应用级协程作用域
├── AlarmEditorActivity.kt            # 新建/编辑闹钟页面
├── AlarmRingingActivity.kt           # 独立锁屏响铃任务与关闭/贪睡操作
├── TimerRingingActivity.kt           # 计时结束时承载原版居中提醒框的独立任务
├── CityPickerActivity.kt             # 世界时钟城市搜索、索引与选择
├── RingtonePickerActivity.kt         # 使用系统公开铃声目录的独立选择页
├── ClockTab.kt                       # 底部标签及资源映射
├── SystemBars.kt                     # 统一 edge-to-edge 与系统栏 Insets
├── custom/
│   ├── AnalogClockHandsView.kt       # 原版表盘、指针及分层进出场动画
│   ├── AdaptiveFrameSequenceView.kt  # 经典拉环回弹序列的 VSYNC 自适应播放
│   ├── AlarmEarAnimationView.kt      # 响铃卡片铃耳序列的高刷自适应播放
│   ├── AlarmRingingPanelView.kt      # 原版响铃卡片进出场、阻尼与上滑关闭
│   ├── AlarmRepeatDaysView.kt        # 星期多选、快速拖选与节假日设置
│   ├── Classic680RulerView.kt        # 经典计时器竖向拉环、回弹与手势
│   ├── CompactAlarmClockView.kt      # 原版响铃浮层的紧凑机械表盘
│   ├── QuickBarEx.kt                 # 城市列表字母索引及展开网格
│   ├── SmallWorldClockView.kt        # 世界时钟列表的小表盘
│   ├── SmartisanSwitchExView.kt      # 原版位图滑块、拖动动画与触觉反馈
│   ├── SmartisanSwitchView.kt        # 锤子风格闹钟开关
│   ├── SmartisanTimePickerView.kt    # 闹钟滚轮选择器
│   ├── TimerRulerView.kt             # 0–180 分钟刻度、阻尼、惯性与吸附
│   └── WorldClockListView.kt         # 城市时钟列表的拖动与选择承载
├── data/
│   ├── AlarmRepository.kt            # 多闹钟 DataStore 与旧数据迁移
│   ├── DirectBootAlarmStore.kt       # 解锁前可读的最小调度快照
│   ├── ActiveAlarmStore.kt           # 当前响铃 occurrence 与短时 lease
│   ├── TimerAlertStore.kt             # 计时提醒 session 与 Direct Boot 状态
│   ├── TimerPreferencesRepository.kt # 计时器样式的 DataStore 持久化
│   ├── WorldCityRepository.kt        # 城市 TSV 与公开 ZoneId 数据读取
│   └── WorldClockStore.kt            # 已添加城市及顺序持久化
├── model/
│   ├── Alarm.kt                      # 闹钟领域模型及系统调度 identity
│   ├── AlarmOccurrenceCalculator.kt  # 星期、时区与 DST 下次触发计算
│   ├── AlarmRepeat.kt                # 周一至周日 bitmask 与本地化展示
│   ├── ClockStates.kt                # 秒表与倒计时单调时间状态机
│   ├── TimerModels.kt                # 计时器样式、运行态与界面状态
│   └── WorldCity.kt                  # 城市搜索、排序与选择规则
├── ui/
│   ├── AlarmPage.kt                  # 多闹钟页面、编辑态与能力提示
│   ├── AlarmViewModel.kt             # 列表状态、最近闹钟与用户意图
│   ├── AlarmListAdapter.kt           # 稳定 ID 的多闹钟列表绑定
│   ├── ClockPage.kt                  # 页面生命周期接口
│   ├── ClockPages.kt                 # 世界时钟与秒表控制逻辑
│   ├── TimerPage.kt                  # 共享表盘、双样式切换与生命周期
│   ├── TimerSurface.kt               # 计时器样式界面契约
│   ├── TimerViewModel.kt             # 单调倒计时状态机与进程恢复
│   ├── ModernTimerSurface.kt         # 横向刻度计时器界面
│   ├── Classic680TimerSurface.kt     # 经典拉环计时器界面
│   ├── Classic680TimerSoundPlayer.kt # 经典拉环机械音效
│   └── WorldClockListAdapter.kt      # 世界时钟列表绑定与原版行布局
├── alarm/                            # 闹钟调度、恢复、通知、Receiver 与响铃服务
├── timer/                            # 计时结束调度、恢复、提醒通知与循环音服务
└── widget/
    ├── ClockBottomBar.kt             # 原版四标签底栏
    ├── ClockContentFrame.kt          # 480dp 手机内容画布与居中约束
    ├── SmartisanMenuDialog.kt        # R2 MenuDialog 风格底部操作面板
    └── SmartisanModalDialog.kt       # Revone 风格居中弹窗公共外壳

app/src/main/res/
├── layout/                           # Activity 与四个主页面 XML
├── drawable*/                        # 原版 PNG、selector 与本地矢量/shape
├── anim/                             # 闹钟编辑页转场
└── font/                             # 恢复的时钟字形

app/src/main/assets/world_cities.tsv  # 城市、国家、别名与 IANA 时区数据

docs/clock-7.1.1-baseline.md           # 原版证据、参数和首批验收基线
docs/alarm-architecture.md             # 多闹钟、Direct Boot 与系统调度不变量
reverse/r2-stock/
├── app/Clock_7.1.1.apk                # 坚果 R2 原厂时钟，核心/最高优先级基准
├── framework/                         # 同机 Android 11 / Smartisan OS 框架证据
    ├── framework-res.apk
    ├── framework.jar
    ├── smartisanos.jar
    └── smartisanos_11.apk
├── decompiled/                        # R2 原厂 JADX/Vineflower/apktool 输出
├── tooling/apktool-frameworks/        # 项目私有的 framework 解码缓存
└── analysis/README.md                 # 逆向目录、证据顺序与关键入口索引

reverse/apk/Clock_7.1.1.apk            # 第三方移植版，仅保留为历史对照
```

## 当前架构与工具链

- **UI**：纯 Kotlin + XML/View，`ClockActivity` 承载四个主页面；城市、闹钟编辑和铃声选择保持独立 Activity。
- **世界时钟**：城市目录来自本地 TSV，时区计算使用 `java.time.ZoneId`；已选城市和排序由 `WorldClockStore` 保存。
- **弹窗**：重复和标签使用 `SmartisanModalDialog` 的 Revone 外壳，内容结构、尺寸和提交语义以原版 Clock 为准；不要退回平台默认 `AlertDialog`。
- **铃声**：使用公开 `RingtoneManager(TYPE_ALARM)` 和 Activity Result API，不恢复移植包中被硬编码的假铃声列表。
- **动画**：`ValueAnimator`、`ViewPropertyAnimator` 与 `postOnAnimation`，遵循系统动画倍率；不要用固定高频线程轮询替代 VSYNC。
- **时间**：秒表与倒计时使用 `SystemClock.elapsedRealtime()` 对应的单调时间，生命周期恢复后从状态重新渲染。
- **闹钟**：`AlarmRepository` 保存多闹钟；所有变更经 `AlarmCoordinator` 串行化后使用 `AlarmManager.setAlarmClock()` 调度，并同步 Direct Boot 快照。触发、响铃页、应用悬浮层、通知和操作以 occurrence identity 做幂等校验。
- **系统 UI**：compile/target API 37，统一通过 `SystemBars.kt` 启用边到边；底栏背景绘制到手势导航区域，交互内容使用 `navigationBars` Insets 避让。不要重新添加黑色 navigation scrim。
- **工具链**：Gradle 9.6.1、AGP 9.2.1、AGP 9 内置 Kotlin、Java 17 字节码目标；Gradle Client/Daemon 统一使用 JetBrains JBR 25。
- **依赖**：AndroidX Core 1.19.0、Activity 1.13.0、Lifecycle 2.11.0、DataStore 1.2.1；测试使用 JUnit 4、AndroidX Test 与 Espresso。
- **SDK**：minSdk 26，targetSdk/compileSdk 37。

## 最新 Android 适配与官方资料

本项目必须面向最新 Android 设备持续适配，不能只复刻旧系统外观。截至 2026-07-11，Android 16（API 36）是当前稳定且主流的适配基线；Android 17（API 37）是最新 Beta，已经达到 Platform Stability，但仍属于会更新发布说明和已知问题的预发布平台。这个状态是时间快照，不得长期照抄；开始相关工作时必须重新核实。

模型训练知识不视为最新 Android API 的可靠来源。凡涉及 Android 16、Android 17 或后续版本，以及可能随平台、SDK、AndroidX、AGP、Gradle、Kotlin 或 Play 政策变化的内容，必须主动联网查询官方一手资料后再设计和编码。优先使用 `android docs search/fetch`，必要时搜索互联网，但技术结论只采用 Android Developers、Android Open Source Project、Google Play、AndroidX、Kotlin 和 Gradle 的官方文档或发布说明，不以第三方博客、搜索摘要或既有记忆作为最终依据。

每次提高 `compileSdk`/`targetSdk`、升级构建工具或触及系统能力前，至少检查并记录与改动相关的官方页面及查阅日期：

- 当前版本的“影响所有应用的行为变更”和“影响目标版本应用的行为变更”；
- SDK 设置、迁移指南、API 差异、发布说明和已知问题；
- 本项目相关的 edge-to-edge、WindowInsets、显示缺口、预测性返回、大屏/折叠屏/多窗口、方向与可调整大小限制；
- 精确闹钟、Full-screen Intent、通知权限、前台服务、后台启动、后台音频、Direct Boot、锁屏与悬浮窗规则；
- AndroidX、AGP、Gradle 和 Kotlin 的兼容矩阵及稳定版发布说明。

当前必须重点遵守的官方结论包括：target API 35 及以上在 Android 15 及以上强制 edge-to-edge；target API 36 时预测性返回默认启用；Android 16 在大屏上可能忽略竖屏、宽高比和不可调整大小声明；target API 37 后不再提供对应的大屏临时退出路径；Android 17 对后台音频交互增加限制，但闹钟音频存在与精确闹钟权限和 `USAGE_ALARM` 相关的专门条件。实现前仍需回到最新官方原文确认，不能只依赖本段摘要。

官方入口：

- [Android 16 行为变更](https://developer.android.com/about/versions/16/behavior-changes-16)
- [Android 17 概览](https://developer.android.com/about/versions/17/)
- [Android 17 行为变更](https://developer.android.com/about/versions/17/behavior-changes-17)
- [Android 17 发布说明](https://developer.android.com/about/versions/17/release-notes)
- [View 系统 edge-to-edge](https://developer.android.com/develop/ui/views/layout/edge-to-edge)
- [View 响应式与自适应布局](https://developer.android.com/develop/ui/views/layout/responsive-adaptive-design-with-views)

项目继续使用 XML/View，不因官方示例优先展示 Compose 就引入 Compose。适配应保持原版 480dp 手机画布的视觉比例，同时让外层容器、Insets、窗口尺寸和状态恢复能够安全应对现代设备；不得依赖锁定竖屏、禁止调整大小或固定宽高比来掩盖布局问题。相关平台行为的完整验收矩阵至少包括 Android 16 稳定镜像、Android 17 最新可用模拟器镜像和真我 GT8 Pro，并明确区分“运行系统版本行为”与“targetSdk 行为”；设备侧测试按下方“构建与验证”的协作分工执行。

## 软件工程与代码品质

代码必须干净、优雅、有品味，并以长期维护为目标，而不是把反编译结果逐句翻译或为通过当前截图堆叠补丁。实现应满足以下约束：

- 修改前先理解数据流、生命周期、线程、状态所有权和模块边界；新增较大功能前先规划文件归属与依赖方向，保持上方项目结构说明同步。
- 类和文件保持单一职责。Activity 负责系统入口与页面编排，View/ViewGroup 负责测量、绘制和输入，状态机与领域计算保持纯 Kotlin，持久化、系统调度和媒体播放通过清晰接口隔离。
- 避免巨型 Activity/View、万能 `Utils`、布尔标志堆叠、重复状态源、跨层回调链、隐式全局状态和散落的魔法数字。复杂状态使用有名字的模型、sealed 类型或明确状态机表达。
- 原版参数集中定义并注明证据来源；资源尺寸、颜色、时序和 selector 优先放在合适的资源或专用常量中，不在多个页面复制近似值。
- API 和命名表达业务意图。注释解释“为什么”和不变量，不复述代码；只为真正需要调用方理解的公共契约编写 KDoc，不用注释掩盖晦涩实现。
- 优先组合、不可变数据、显式依赖和小而可测试的函数；谨慎抽象，出现稳定重复或明确边界时再提取，不为“架构感”增加空层、样板和无收益依赖。
- 时间、时区、系统时钟、调度和外部能力应可注入或可替换，测试不得依赖真实当前时间、执行顺序巧合、网络或设备残留状态。
- 协程必须有明确作用域、取消和错误策略；View 动画及逐帧工作跟随生命周期和 VSYNC，不在绘制热路径分配对象，不用后台轮询弥补状态设计问题。
- 每次重构保持行为可验证；修复根因而不是增加特例。提交前删除死代码、调试开关、无用资源和过期注释，但不得顺手改动与任务无关的用户代码。
- 新文件应放入最窄且语义明确的包；当一个目录开始混合 UI、领域、存储和平台接入职责时，先规划拆分再继续扩展，并同步更新 `AGENTS.md` 和相关架构文档。

## 核心参考基准

从现在起，坚果 R2 上的原厂产物是本项目唯一的核心依据。此前使用的
`reverse/apk/Clock_7.1.1.apk` 是第三方移植版，移植者已明确说明其还原并不完善；该 APK、现有
JADX/apktool 输出以及由它整理出的基线只能用于追溯历史实现，不能继续作为原版事实来源。

### 证据优先级

1. `reverse/r2-stock/app/Clock_7.1.1.apk` 的代码、资源和 Manifest。
2. 同机提取的 `reverse/r2-stock/framework/`，用于正确解析 OEM 资源、属性、控件和 Smartisan OS 私有 API 的原始语义。
3. 用户提供的原版截图及其他来源明确、可复查的视觉资料。
4. 坚果 R1 上旧版拉条时钟的实际行为，仅用于经典拉条计时器及可确认的 Smartisan 机械交互，不得替代 R2 版本证据。
5. `reverse/apk/Clock_7.1.1.apk` 及既有反编译目录仅作为第三方移植历史对照；与前四项冲突时必须舍弃其结论。

已有源码、资源和 `docs/clock-7.1.1-baseline.md` 不因已经实现而自动视为正确。后续触及相关页面或行为时，必须先用 R2 原厂产物重新核验；发现差异应修正实现并同步更新基线文档。

### 坚果 R2 原厂文件

- 应用：Smartisan Clock 7.1.1
- 包名：`com.smartisanos.clock`
- versionCode：102
- compileSdk：28，minSdk：24，targetSdk：25
- APK：`reverse/r2-stock/app/Clock_7.1.1.apk`
- APK SHA-256：`7af83b54309cd8d67add398003cfcf8adae6f43f127b2a6722f17a507d38342a`
- 签名证书 SHA-256：`99cb9a0ece39c4301e22150e5d7238ee9b4073042054c60baafd68f3a7c57574`（证书主体 `O=Smartisan`）
- `framework-res.apk` SHA-256：`9dd895fdcba12cf1983a014bab794a93200e7846c1bf6c9233c0c6d6830d0c31`
- `framework.jar` SHA-256：`4f330a8f8383ae174d17df9a4ec1d78b8131aaa01e39bb6a6ac4d36f2162eb27`
- `smartisanos.jar` SHA-256：`73f75f1391d581ec0457562047bea99a61b75387b7782b6aba44f442987deaa7`
- `smartisanos_11.apk` SHA-256：`121a762867be00a65c3781e30c6d14f44ffe64f0f1281b75d90589a8289ebe42`

### 可用测试设备与职责

- **坚果 R1**：安装的是旧版拉条时钟，可用于核对经典拉条计时器的布局、手势、阻尼、回弹、声音与机械触感；它不是 R2，不得用其余页面的视觉或行为覆盖 R2 原厂 APK 证据。
- **真我 GT8 Pro**：当前复刻应用的主力测试机，用于验证现代 Android 下的安装、启动、Insets、手势导航、闹钟调度、通知、锁屏/悬浮入口、性能和进程恢复；它只能证明复刻版兼容性，不能证明 R2 原版行为。
- 当前没有坚果 R2 真机。R2 原版结论以已提供的 APK、配套系统框架和可复查截图为准。

旧第三方移植 APK 位于 `reverse/apk/Clock_7.1.1.apk`，SHA-256 为
`c586ccc1f8dd0102c0782e17cfdc704bf5c247547ec8a43a172b774181fa4aac`，其 targetSdk 为 30。不要从该移植版反推 R2 原版行为。

不要凭印象重新设计原版行为。涉及表盘、指针、字体、颜色、按钮位移、计时器阻尼或页面切换时，先检查 R2 原厂 APK 和配套框架，再按需使用 JADX、apktool、资源表及用户提供的原版截图补证据。

## 已恢复的关键体验

- 底部顺序为世界时钟、闹钟、秒表、计时器，默认进入闹钟；页面保留原版离场后切换和分层入场节奏。
- 世界时钟与闹钟表盘恢复时针、分针、秒针、阴影、中心盖帽和 700/900/1000ms 指针入场。
- 世界时钟已支持城市搜索、中英文别名、字母索引网格、IANA 时区、城市增删和拖动排序。
- 秒表支持开始、暂停、继续、记圈、重置以及机械按钮按压位移，状态由单调时间驱动。
- 计时器恢复 0–180 分钟横向刻度、原版三段阻尼、惯性、整分钟吸附、越界回弹、触觉反馈和自动开始语义。
- 计时器支持持久化选择经典拉环或横向刻度，默认使用经典拉环；两种样式共享 7.1.1 表盘并保留各自的分层进出场动画。
- 计时结束由独立精确调度与 session identity 仲裁；提醒页使用 R2 居中“提醒”卡片，原版 `timer.ogg` 延迟 600ms 后循环，确认或 20 分钟超时即停止，并通过公开 Full-screen Intent、锁屏 Activity 与可选悬浮层适配现代后台限制。
- 闹钟编辑页支持原版七日多选与快速拖选、法定节假/调休日开关、系统铃声选择、标签编辑和本地保存。
- 闹钟支持多条列表、精确调度、重启与改时恢复、Direct Boot、全屏响铃、其他应用上层响铃、通知操作、十分钟贪睡及一次性闹钟自动关闭。
- 响铃页恢复原版中央机械闹钟卡片、背景压暗、铃耳序列、顶部进出场、下拉阻尼与上滑关闭；锁屏入口使用现代 Full-screen Intent 和公开锁屏 API，已解锁场景在用户授权后使用公开 `TYPE_APPLICATION_OVERLAY`。
- 重复与标签弹窗使用统一 Revone 外壳；节假日滑块保留原版绿色位图、拖动动画和用户操作震动。
- 主页面、城市选择、闹钟编辑和铃声选择页已经统一适配透明系统栏、刘海和手势导航，浅色背景使用深色系统栏图标。

## 尚未完成

- “跟随法定节假、调休日”目前完成界面与状态保存；原版依赖 Smartisan 日历私有 Provider，未来调度时必须接入公开或项目自有的节假日数据源，不得复制 OEM 私有接口。
- 秒表圈次的完整列表体验与长期后台语义仍需补齐。
- 需要继续以 R2 原版资源、用户提供的截图和模拟器校准字体基线、阴影及动画节奏；经典拉条的按钮触感可在坚果 R1 上核对，其余无法确认的 R2 触感和刷新率差异应明确标为待验证，不得凭印象补全。
- 平板、折叠屏、多窗口和三键导航仍需补充模拟器矩阵验证；真我 GT8 Pro 作为主力兼容性设备，但不得据此声称已经覆盖所有厂商设备。

## 构建与验证

```bash
./gradlew testDebugUnitTest
./gradlew assembleDebug
./gradlew lintDebug
./gradlew assembleRelease
```

提交页面、动画、触摸或 Insets 改动前，至少运行：

```bash
./gradlew testDebugUnitTest assembleDebug lintDebug
```

默认协作分工如下：Codex 负责代码审查、静态检查以及上述 Gradle 自动化验证；用户负责模拟器和真机的安装、截图、日志采集与实际交互测试，并将结果反馈给 Codex。除非用户明确要求，Codex 不主动连接、安装或操作模拟器、真我 GT8 Pro 或坚果 R1。设备测试尚未反馈时，应在交付说明中明确列出待验收项，不得声称已经完成对应的真机或交互验证。

涉及 View 尺寸、表盘绘制、指针动画、计时器拖拽、系统 Insets 或 Activity 转场时，仍须由用户在模拟器或真我 GT8 Pro 上通过截图、日志和实际交互验收；经典拉条改动还应由用户在坚果 R1 上对照旧版行为。不能把“编译通过”或“静态截图接近”当作最终设备验收。

验证交互时至少覆盖：按下与取消、快速切页、前后台恢复、系统关闭动画、秒表暂停/继续、计时器越界与吸附、闹钟保存与返回，以及手势导航区域是否无黑边且不遮挡触控目标。

## Git 约定

- 初始空项目提交已经完成；后续提交统一使用简体中文 Conventional Commits。
- 首行格式示例：`feat(clock): 恢复计时器交互`。
- 首行后空一行，正文使用短横线逐项说明具体变更。
- `reverse/`、构建目录、APK/AAB、IDE 配置和本机配置不得提交。
- 未经明确要求不要修改、重写或压缩已有提交历史。

---
> Source: [Mangi-11/SmartisanClock-Revived](https://github.com/Mangi-11/SmartisanClock-Revived) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
