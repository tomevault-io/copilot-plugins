## lumitv

> - **名称**: LumiTV (流光)

# AIVision 项目开发指南

## 项目概览
- **名称**: LumiTV (流光)
- **平台**: Android TV (API 30+)
- **技术栈**: Flutter, Riverpod 3, Hooks, Generator
- **目标**: 打造一款画质优先和功能全面的电视影音播放器。
- **语言**: 以后请用中文跟我交流并且项目中也要求中文优先

## 常用开发命令
- **生成代码**: `dart run build_runner build --delete-conflicting-outputs`
- **持续监控生成**: `dart run build_runner watch --delete-conflicting-outputs`
- **运行应用**: `flutter run`
- **清理缓存**: `flutter clean && flutter pub get`
- **连接 TV 模拟器/真机**: `adb connect <IP_ADDRESS>`

## 代码规范与约束
### 1. 状态管理 (Riverpod)
- 必须使用 **Riverpod Generator** (`@riverpod` 注解)。
- 尽量使用异步 Provider (`FutureProvider` 风格) 处理 API 请求。
- 提供者文件命名：`xxx_provider.dart`。

### 2. UI 逻辑 (Hooks)
- 页面组件优先使用 `HookConsumerWidget`。
- 简单的生命周期管理（如 `TextEditingController` 或 `FocusNode`）必须使用 `useTextEditingController` 或 `useFocusNode`。

### 3. Android TV 适配特性
- **焦点管理**: 所有交互组件必须处理 `onFocusChange` 和 `onKeyEvent`。
- **视觉反馈**: 遥控器聚焦时，组件需有明显的放大 (Scale) 或发光效果。
- **导航**: 使用方向键 (D-pad) 进行逻辑循环，避免死循环焦点。

### 4. 命名约定
- 类名：`PascalCase` (如 `MovieCard`)
- 变量/函数：`camelCase` (如 `fetchMovieList`)
- 文件名：`snake_case` (如 `movie_detail_screen.dart`)

## 核心依赖参考
- 状态管理: `hooks_riverpod`, `riverpod_annotation`
- UI 辅助: `flutter_hooks`
- 图标: `material_design_icons_flutter` (可选)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhangheng2022) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
