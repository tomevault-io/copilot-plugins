## commandgathering

> - 这是 macOS 原生 SwiftUI 桌面工具，用于管理常用命令并在 App 内部终端 tab 中执行。

# Command Gathering 工作规则

## 项目概况
- 这是 macOS 原生 SwiftUI 桌面工具，用于管理常用命令并在 App 内部终端 tab 中执行。
- 当前工程是 Swift Package，入口 target 是 `CommandGatheringApp`，核心逻辑在 `CommandGatheringCore`。
- 终端组件使用项目本地 vendored `Vendor/SwiftTerm`，因为 SwiftPM 远程拉取 GitHub 在当前网络下不稳定。

## 命令与验证
- 运行测试：`rtk swift test`
- 调试构建：`rtk swift build`
- 运行 App：`rtk swift run CommandGatheringApp`
- 打包 `.app`：`rtk scripts/build-app.sh`
- 打包 GitHub Release 干净压缩包：`rtk scripts/build-release.sh`

## 数据存储
- `swift run` 时，命令配置写入当前工作目录：`CommandGatheringData/commands.json`
- 打包 `.app` 运行时，命令配置写入：`CommandGathering.app/CommandGatheringData/commands.json`
- `scripts/build-app.sh` 重建 `dist/CommandGathering.app` 时会保留已有 `CommandGatheringData`。
- `scripts/build-release.sh` 必须走独立输出路径，生成 `dist/release/CommandGathering-Clean.app` 及其 zip；不要覆盖用户正在使用的 `dist/CommandGathering.app`，也不要把本地带数据的 `.app` 直接当发布资产。
- 不要把用户运行后生成的 `CommandGatheringData` 当成源码改动提交。
- `CommandStore` 的迁移逻辑必须尊重用户对默认命令的删除、改名和编辑；不要在每次启动时重新补齐默认命令。只有检测到旧默认 `git status --short` 时才做一次旧配置迁移。

## 开发注意事项
- 命令按钮和终端 tab 的绑定逻辑在 `TerminalCoordinator`，先改测试再改实现。
- 点击命令的规则是：没有绑定 tab 时新建并执行一次；已有绑定 tab 时只切换，不重复执行。
- 右侧命令卡片支持右键菜单，至少包含编辑和删除。
- 右侧顶部同时保留“新建命令”和“临时命令”：新建命令写入当前分组配置，临时命令只打开临时终端且不保存配置。
- 右侧顶部有齿轮设置按钮，点击后打开独立设置界面；不要再把设置项直接塞在齿轮下拉菜单里。当前设置界面先保留“终端”分区和“终端跟随分组”开关，状态写入 `UserDefaults.terminalFollowsSelectedGroup`；后续新增设置继续放入该设置界面。开启后，命令终端归属命令所在分组，临时终端和空白终端归属创建时的当前分组，终端 tab 栏只显示当前分组对应终端，但切换分组不能关闭或重启其他分组的终端。从关闭切到开启时，已有未归属分组的终端归到当前选中分组。
- 侧边栏显隐按钮放在左侧栏 `命令分组` 标题右边，只显示图标不要额外背景包裹，状态写入 `UserDefaults.isSidebarHidden`；隐藏后用最左侧窄恢复条显示回来，不要把显隐按钮放到右侧分组标题前。
- 左侧 `命令分组` 和右侧当前分组标题下不要显示说明小字。
- 右侧顶部靠右有暗亮模式图标按钮，状态写入 `UserDefaults.themeMode`。
- 右侧命令卡片区域空白处支持右键新建命令；如果 SwiftUI `contextMenu` 被 `ScrollView` 吃掉，使用 AppKit 透明空白层只覆盖卡片实际内容下方空白区域。
- 新建/编辑命令的校验错误必须在编辑弹窗内直接显示，不能只依赖根视图 alert，否则 sheet 覆盖时会像保存无反应。
- 右侧命令卡片区域和终端区域之间支持上下拖动，只保留终端顶部细分隔条，不要在命令卡片和终端 tab 之间额外制造大块留白；拖动后的命令卡片区域高度写入 `UserDefaults.commandListHeight`。
- 非当前终端 tab 的标题必须在暗亮模式下都可读，不能使用过低对比度文本。
- 终端 tab 栏保持紧凑，高度和右侧 `+` 按钮接近，不要保留大块上下空白。
- 左侧分组支持拖拽排序、编辑名称和删除，排序写入 `sortOrder` 并持久化；删除分组会同步删除该组下命令，至少保留一个分组。
- 终端会话默认启动用户 shell 的交互模式，让 `~/.zshrc` 等交互配置生效。
- App 不强行配置 zsh 语法高亮；输入高亮由用户自己的 `~/.zshrc` 决定。

---
> Source: [wa3ytsm/CommandGathering](https://github.com/wa3ytsm/CommandGathering) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-18 -->
