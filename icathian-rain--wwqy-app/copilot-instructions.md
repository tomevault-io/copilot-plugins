## wwqy-app

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 语言偏好

- 所有回复、注释说明和新增文档默认使用中文。

## 常用命令

### 依赖与运行

- 安装依赖：`flutter pub get`
- 启动应用：`flutter run`
- 指定设备启动：`flutter run -d windows` / `flutter run -d chrome`
- 列出可用设备：`flutter devices`

### 质量检查

- 静态检查：`flutter analyze`
- 运行全部测试：`flutter test`
- 运行单个测试文件：`flutter test test/widget_test.dart`
- 按名称运行单个测试：`flutter test --plain-name "App should render"`

### 构建

- Windows 桌面构建：`flutter build windows`
- Web 构建：`flutter build web`
- Android APK 构建：`flutter build apk`

### 数据与调试

- 本项目使用本地 SQLite，数据库文件名为 `wwqy_app.db`。
- 需要重置预置数据时，删除本机数据库文件后重新启动应用即可触发建库和预置数据写入。
- Windows / Linux 桌面端在 [lib/main.dart](lib/main.dart) 中通过 `sqflite_common_ffi` 初始化数据库工厂。

## 项目架构

这是一个 Flutter 本地点位记录应用，当前主要围绕《无畏契约》点位数据展开。整体是典型的单应用 + Provider 状态管理 + SQLite 本地持久化结构。

### 启动与应用入口

- [lib/main.dart](lib/main.dart) 负责应用启动，以及 Windows / Linux 桌面端的 `sqflite_common_ffi` 初始化。
- [lib/app.dart](lib/app.dart) 创建根级 `ChangeNotifierProvider<LineupProvider>`，并配置全局 `MaterialApp` 主题。
- Windows 中文字体兼容也是在 [lib/app.dart](lib/app.dart) 中处理，修改主题时不要破坏该逻辑。

### 数据流主线

UI 不直接访问数据库，而是遵循：

`Screen -> LineupProvider -> LineupRepository -> DatabaseHelper(SQLite)`

对应职责如下：

- `screens/`：页面与交互。
- [lib/providers/lineup_provider.dart](lib/providers/lineup_provider.dart)：维护当前页面所需状态，包括游戏、地图、特工、点位列表以及筛选条件。
- [lib/data/lineup_repository.dart](lib/data/lineup_repository.dart)：封装数据库查询与写入，不承担 UI 状态。
- [lib/data/database_helper.dart](lib/data/database_helper.dart)：单例数据库入口，负责建表与首次初始化数据。

如果要改列表刷新、筛选逻辑或新增字段，通常需要同时改 `provider + repository + database schema + model`。

### 导航与页面结构

应用当前是逐级导航流：

1. [lib/screens/home_screen.dart](lib/screens/home_screen.dart)：加载游戏列表。
2. [lib/screens/map_select_screen.dart](lib/screens/map_select_screen.dart)：按游戏展示地图。
3. [lib/screens/lineup_list_screen.dart](lib/screens/lineup_list_screen.dart)：按地图展示点位列表，并在页面内维护筛选交互。
4. [lib/screens/add_lineup_screen.dart](lib/screens/add_lineup_screen.dart)：新增点位，选择特工 / 攻防 / 包点 / 标题 / 说明 / 图片。
5. [lib/screens/lineup_detail_screen.dart](lib/screens/lineup_detail_screen.dart)：查看点位详情、图片轮播、全屏查看、删除。

几个页面都依赖 `LineupProvider` 的异步加载方法，常见模式是在 `initState` 中通过 `addPostFrameCallback` 触发加载。

### 数据模型与存储结构

`lib/models/` 中的模型基本与 SQLite 表一一对应：

- `Game`
- `GameMap`
- `Agent`
- `Lineup`
- `LineupImage`

数据库定义位于 [lib/data/database_helper.dart](lib/data/database_helper.dart)，当前核心表有：

- `games`
- `maps`
- `agents`
- `lineups`
- `lineup_images`

注意点：

- `lineup_images` 通过 `lineup_id` 关联到 `lineups`。
- 仓储层删除点位时，会先删 `lineup_images` 记录，再删 `lineups`。
- Provider 在删除点位后还会继续删除磁盘上的实际图片文件，因此删除逻辑分布在 repository 和 provider 两层，不要只改其中一层。

### 预置数据机制

- 预置游戏、地图、特工数据位于 [lib/data/preset_data.dart](lib/data/preset_data.dart)。
- 首次建库时，`DatabaseHelper._insertPresetData()` 会把这些静态数据写入 SQLite。
- 当前项目只内置《无畏契约》数据，地图和特工名称使用中文。

如果新增游戏或补充地图/特工，优先更新 `PresetData`，再确认列表页加载逻辑是否仍适用。

### 图片处理

- 图片选择与复制由 [lib/utils/image_helper.dart](lib/utils/image_helper.dart) 负责。
- 选中的图片不会直接依赖原路径，而是复制到应用文档目录下的 `lineup_images/` 子目录。
- 数据库存储的是复制后的目标路径。

这意味着：

- 调整图片选择逻辑时，要保留“复制到应用私有目录”这一步。
- 删除点位时，除了删数据库，还要删应用目录中的图片文件。

### 平台相关约束

- Windows / Linux 桌面端需要 `sqflite_common_ffi`，入口初始化不要删。
- `image_picker` 的 `ImageSource.camera` 不能在当前 Windows 桌面实现中直接使用，相关页面应只在 Android / iOS 显示拍照入口。
- 当前主题是暗色 Material 3；如果做 UI 美化，优先在现有主题基础上调整，不要引入未经确认的新 UI 库。

## 测试现状

- 当前只有一个基础 Widget 测试文件：[test/widget_test.dart](test/widget_test.dart)
- 该测试只验证根应用能渲染标题文本。
- 如果修改了应用入口、Provider 注入方式或首页标题，需要同步更新该测试。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Icathian-Rain)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Icathian-Rain)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
