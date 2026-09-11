## daro

> > 本文件为 AI 智能体（及人类开发者）提供项目导航与编码规范。

# AGENTS.md

> 本文件为 AI 智能体（及人类开发者）提供项目导航与编码规范。
> 遵循本文档可确保代码风格一致、主题正确、组件归属清晰。

---

## 项目概览

**daro** 是一个用 Flutter 构建的 桌面数据库管理工具。
目标是高信息密度、桌面级交互体验，支持明 / 暗双主题。

| 维度 | 说明 |
|---|---|
| 框架 | Flutter (Dart SDK ≥3.0) |
| 状态管理 | `provider` + `ChangeNotifier` (AppState) |
| 主题 | 自建 `AppPalette` / `Tokens` 语义色板 + `base_ui_flutter` 的 `DesktopTokens` / `TokenScope` |
| UI 组件库 | `base_ui_flutter`（本地 path 依赖，git submodule） |
| 字体 | `chinese_font_library` 中文排版优化 |

---

## 目录结构

```
daro/
├── lib/
│   ├── main.dart                 # 应用入口；TokenScope 包裹；主题切换
│   ├── app/
│   │   └── app_state.dart        # AppState: 主题模式、标签管理、表选择
│   ├── data/
│   │   └── db_data.dart     # 静态模拟数据（连接 / 数据库 / 表列表）
│   ├── pages/
│   │   └── main_page.dart     # 主页面布局：顶栏 + Ribbon + 三栏 + 状态栏
│   ├── theme/
│   │   └── app_theme.dart        # AppPalette 明暗色板 + Tokens.of() + toDesktopTokens() 桥接
│   └── widgets/                  # 应用级 widget（业务耦合，不可复用）
│       ├── top_menu.dart         # 顶部菜单栏(复用 base-ui-flutter MenuStrip) + 主题切换按钮
│       ├── ribbon.dart           # 工具栏
│       ├── database_tree.dart    # 左侧连接树
│       ├── object_panel.dart    # 中部对象面板（108 表网格）
│       ├── object_tabs.dart      # 对象路径标签
│       ├── view_tabs.dart        # 视图标签栏
│       ├── database_info.dart    # 右侧详情面板
│       ├── status_bar.dart       # 底部状态栏
│       ├── table_icon.dart       # 表图标（应用级，不归入 base-ui-flutter）
│       ├── table_data_page.dart  # 表数据浏览页
│       └── query_page.dart       # SQL 查询编辑页
│
├── base-ui-flutter/             # 独立 UI 组件库（git submodule）
│   └── lib/
│       ├── base_ui_flutter.dart  # barrel export（单一入口）
│       └── src/
│           ├── foundation/       # DesktopTokens、TokenScope、Control 基类
│           ├── common/            # Button、Input、Label、CheckBox、ComboBox…
│           ├── lists/             # ListBox、TreeView、DataGridView…
│           ├── containers/         # GroupBox、TabControl、SplitContainer…
│           ├── menus/             # MenuStrip、ToolStrip、StatusStrip…
│           ├── overlay/           # Popover、MessageBox、Toast…
│           ├── dialogs/           # ColorDialog、DateTimePicker…
│           ├── data/              # Chart、Pagination…
│           ├── scroll/            # ScrollBar、TrackBar
│           └── misc/              # ProgressBar、Skeleton、Spinner…
│
├── pubspec.yaml                  # base_ui_flutter 通过 path 依赖引入
└── AGENTS.md                     # 本文件
```

---

## 双层主题系统

### 第一层：AppPalette / Tokens（应用语义色板）

定义在 `lib/theme/app_theme.dart`。这是应用的**主**主题系统，包含 应用特有的语义色（`connIcon`、`schemaIcon`、`folderIcon`、`tableIcon` 等）。

```dart
// 取色方式
final t = Tokens.of(context);
Container(color: t.surface);
Text('hello', style: TextStyle(color: t.textPrimary));
```

### 第二层：DesktopTokens / TokenScope（base_ui_flutter 令牌）

`base_ui_flutter` 组件库使用自己的 `DesktopTokens` 系统。通过 `AppPalette.toDesktopTokens()` 桥接方法，将应用色板映射到 `DesktopTokens` 字段，使 `base_ui_flutter` 组件自动跟随应用主题。

```dart
// 在 main.dart 中
home: Builder(
  builder: (context) {
    final brightness = Theme.of(context).brightness;
    final palette = brightness == Brightness.dark
        ? AppTheme.dark
        : AppTheme.light;
    return TokenScope(
      tokens: palette.toDesktopTokens(),
      child: const MainPage(),
    );
  },
),
```

### 桥接映射

| AppPalette 字段 | DesktopTokens 字段 | 用途 |
|---|---|---|
| `selectedBg` | `primaryColor` | 选中 / 焦点色 |
| `menuBar` | `backgroundColor` | 窗口背景 |
| `textPrimary` | `foregroundColor` | 主文字 |
| `border` | `borderColor` | 控件边框 |
| `surface` | `surfaceColor` / `cardColor` | 面板背景 |
| `ribbonBar` | `controlColor` | 按钮 / 工具栏 |
| `popupBg` | `popoverColor` | 弹出菜单 |
| `textSecondary` | `mutedForegroundColor` | 次要文字 |
| `tabBar` | `secondaryColor` | 标签栏 |
| `gutterBg` | `mutedColor` | 行号槽 |
| `textOnSelected` | `accentForegroundColor` | 选中态文字 |

> **新增 DesktopTokens 字段时**：在 `toDesktopTokens()` 中补充映射。

---

## 组件归属规则

### ⚠️ 强制规则：禁止在 daro 中创建自定义 UI 组件

**任何 UI 组件需求，必须遵循以下流程：**

1. **优先检查 `base-ui-flutter`**：查看是否已有该组件（见底部组件速查表）
2. **存在 → 直接使用**：从 `base_ui_flutter` barrel 导入使用
3. **不存在 → 在 `base-ui-flutter` 中创建通用组件**：按下方步骤新增，然后在 daro 中引入
4. **绝对禁止**：在 `lib/widgets/` 或任何 daro 业务代码中自造 UI 组件（如 `_FormField`、`_CloseButton`、`_ApplyButton` 等）

> **违规示例**：主题定制弹窗中曾出现 `_ApplyButton` 自定义组件，应直接使用 base-ui 的 `Button`。

### ⚠️ 强制规则：禁止直接使用 Material 组件

**所有 UI 控件必须使用 `base_ui_flutter` 封装组件，禁止直接使用 Flutter Material 控件**（`ElevatedButton`、`TextField`、`Checkbox`、`Dialog`、`SnackBar`、`Tooltip`、`InkWell`、`Material` 等）。

| 场景 | ✅ 使用 base-ui 组件 | ❌ 禁止的 Material 组件 |
|---|---|---|
| 按钮 | `Button` | `ElevatedButton` / `TextButton` / `FilledButton` / `IconButton` |
| 输入 | `Input` | `TextField` / `TextFormField` |
| 勾选 | `CheckBox` | `Checkbox` / `Switch` / `CheckboxListTile` |
| 下拉 | `ComboBox` | `DropdownButton` / `DropdownMenu` |
| 弹窗 / 提示 | `MessageBox` / `Popover` / `Toast` | `Dialog` / `AlertDialog` / `SnackBar` |
| 分组 | `GroupBox` | `Card` / `ExpansionTile` |
| 标签页 | `TabControl` | `TabBar` / `TabBarView` |
| 点击反馈 | `GestureDetector` / `Listener` / `MouseRegion` | `InkWell` / `InkResponse` |

> **base-ui 未提供的组件**：一律按上述流程在 `base-ui-flutter` 中创建通用组件，禁止直接用 Material 组件替代。
> **布局基础组件**（`Row`、`Column`、`Stack`、`Container`、`ListView`、`GridView`、`CustomPaint` 等）不属于 Material 控件，可正常使用。
> **例外：查询页 SQL 编辑器**采用第三方包 `re_editor`（用户决策豁免 base-ui 组件禁令，见 `query_page.dart`）。其自带的 `CodeEditor` / 行号 / 语法高亮 / 补全框架不重复造轮子；但补全弹层视图（`_SqlPromptPanel`）仍遵循零延迟交互与 token 取色规范。

### `lib/widgets/` 仅允许页面级组合

`lib/widgets/` 中的文件只能是**页面级组合 widget**，满足以下全部条件：
- **业务耦合**：直接依赖 `AppState`、`DbData` 或应用数据模型
- **布局特定**：为 daro 的特定布局而设计，不可独立复用
- **组合性质**：由多个 base-ui 子组件组合而成，自身不包含独立 UI 原语

当前示例：`TopMenu`、`Ribbon`、`DatabaseTree`、`ObjectPanel`、`ViewTabs`、`StatusBar`、`QueryPage`、`TableDataPage`、`DatabaseInfo`、`ObjectTabs`、`TableIcon`

> **注意**：`TableIcon` 是特例（应用主题专用，绑定 `Tokens.of(context).tableIcon` 与 108 表项性能优化），不迁移到 base-ui。

### 放入 `base-ui-flutter`（可复用组件库）

满足以下全部条件：
- **无业务依赖**：不引用 `AppState`、`DbData` 或任何应用层代码
- **Token 驱动**：所有视觉值通过 `DesktopTokens` 或构造参数传入，零硬编码颜色 / 字体 / 间距
- **独立可用**：可在任意 Flutter 项目中直接使用，无需修改
- **有明确语义**：代表一个通用的 UI 控件或原语

> 注意：`TableIcon` 是 daro 应用级组件（绑定 `Tokens.of(context).tableIcon` 主题色与 108 表项性能优化），**不迁移到** `base-ui-flutter`。

#### 新增 base-ui-flutter 组件的步骤

1. 在 `base-ui-flutter/lib/src/<命名空间>/` 下创建 `.dart` 文件
2. 实现 widget，遵循 headless + token 驱动约定：
   - 接受可选 `tokens` 参数（`DesktopTokens?`）
   - 取色链：`tokens ?? TokenScope.maybeOf(context) ?? DesktopTokens.winForm`
   - 不硬编码任何颜色 / 字体 / 间距
3. 在 `base_ui_flutter.dart` barrel 中添加 `export`
4. 在 `CHANGELOG.md` 中记录
5. 在 `example/lib/pages/` 中添加演示页（可选但推荐）

---

## 视觉延迟 / 特效禁令（用户强烈要求，必须遵守）

**背景**：多次反馈"首次点击/选中慢半拍、点击特效、动画慢、菜单项黄线/加粗"。以下模式一律禁止：

1. **禁止 `onTap` 与 `onDoubleTap` 同时注册在同一手势目标**（InkWell / GestureDetector 都不行）：单击会被双击判定窗口 hold 约 300ms → 视觉延迟。
   ✅ 正确模式：**选中/常用操作 → `Listener.onPointerDown`（按下瞬间触发，零延迟）**，双击动作单独放 `GestureDetector.onDoubleTap`。
   参考：`object_panel.dart` 表项、`connection_dialog_page.dart` `_GridCard`/`_ListRow`、`database_tree.dart` `_node`、base-ui `list_view.dart` `_buildRow`。

2. **禁止用 `window_manager.DragToMoveArea` 包裹可点击区域**：其自带 `onDoubleTap`（双击最大化）→ 内部控件单击被 hold ~300ms（顶部菜单首次展开延迟的根因）。
   ✅ 改用 `GestureDetector(onPanStart: () => windowManager.startDragging())`（无双击判定）。

3. **禁止 `InkWell` / Material 水波纹特效**（用户明确反感"点击特效"）：用 `GestureDetector` / `Listener` / `MouseRegion` 手绘 hover / pressed。
   （残留待统一替换：`status_bar.dart` `_PanelToggleIcon`、`view_tabs.dart`、`table_data_page.dart`、`database_page.dart` 表卡片、`database_tree.dart` 表节点）

4. **禁止带勾选/点击动画的控件**：CheckBox 已重写为自绘无动画（WinForms 快节奏）。新增/重写组件默认无动画。

5. **菜单/浮层性能**：
   - MenuStrip 下拉面板**禁止 `Material(elevation)`**（阴影首次计算 / shader 编译 → "首次展开慢、之后快"）。
   - **禁止依赖 Material 的 `DefaultTextStyle` 兜底**：去掉 Material 后 Text 会继承应用级带 `decoration: underline/double/yellow` + `fontWeight: bold` 的样式（导致菜单项双黄线、加粗）。凡脱离 Material 的文本必须显式 `decoration: TextDecoration.none` + `fontWeight: FontWeight.w400`。
   - MenuStrip 顶层项高亮用 `Row(crossAxisAlignment: stretch)` 填满整行，避免 hover 窄带像"横线"。

6. **hover / 选中色必须明暗自适应**：基于目标底色派生（暗色提亮、亮色加深），禁止直接用 winForm 亮色默认值（暗色下突兀/不可见）。

---

## 主题切换

`AppState` 提供 `themeMode`（默认 `ThemeMode.system`）和 `cycleThemeMode()`。
顶部菜单栏右侧有主题切换按钮，在 **系统 → 明亮 → 暗黑** 之间循环。

顶部菜单栏（`lib/widgets/top_menu.dart`）本身由 base-ui-flutter 的 `MenuStrip` 提供，
原生支持「点击展开 + 菜单打开时鼠标移到其它菜单自动切换」的桌面菜单行为，
并通过 `AppPalette.toDesktopTokens()` 桥接跟随明/暗主题；本文件只负责装配假项目
菜单数据与右侧主题切换按钮。

```dart
// 代码中切换主题
context.read<AppState>().cycleThemeMode();

// 或直接设置
context.read<AppState>().setThemeMode(ThemeMode.dark);
```

---

## 编码规范

### 颜色
- **应用 widget**：通过 `Tokens.of(context)` 取色，不硬编码 `Color(0xff…)`
- **base-ui-flutter 组件**：通过 `DesktopTokens` 取色，不接受 `Color` 参数
- 例外：Ribbon / QueryPage 的功能强调色（运行=绿、停止=红等）是主题无关的中调色，可直接定义

### 性能
- `ObjectPanel` 的 108 表项使用 `ValueListenableBuilder` + 按项通知器，避免全量重建
- `TableIcon` 按颜色缓存 painter，支持 100+ 图标共享同一 `CustomPainter` 实例
- 列表使用 `ListView.builder` + `itemExtent` 懒加载

### 命名
- 应用 widget：PascalCase，按功能命名（`TopMenu`、`DatabaseTree`）
- base-ui-flutter 组件：PascalCase，按 WinForm 语义命名（`Button`、`MenuStrip`、`ToolStrip`）

---

## 构建与验证

```bash
# 静态分析
flutter analyze

# 运行
flutter run -d windows

# base-ui-flutter 单独分析
cd base-ui-flutter && flutter analyze
```

---

## base-ui-flutter 组件速查

| 需求 | 组件 | 命名空间 |
|---|---|---|
| 菜单栏 | `MenuStrip` | menus |
| 右键菜单 | `ContextMenuStrip` | menus |
| 工具栏 | `ToolStrip` | menus |
| 状态栏 | `StatusStrip` | menus |
| 按钮 | `Button` | common |
| 输入框 | `Input` | common |
| 标签 | `Label` | common |
| 复选框 | `CheckBox` | common |
| 下拉框 | `ComboBox` | common |
| 表单字段 | `Field` | common |
| 标签页 | `TabControl` | containers |
| 分割容器 | `SplitContainer` | containers |
| 拖动改宽(独立分隔条) | `Splitter`（三栏/停靠面板：只上报像素增量，宽度由宿主状态掌控） | containers |
| 分组框 | `GroupBox` | containers |
| 树形视图 | `TreeView` | lists |
| 数据网格 | `DataGridView` | lists |
| 列表框 | `ListBox` | lists |
| 进度条 | `ProgressBar` | misc |
| 弹出层 | `Popover` | overlay |
| 对话框 | `MessageBox` | overlay |
| 提示 | `Toast` | overlay |
| 头像 | `Avatar` | misc |
| 面包屑 | `Breadcrumb` | misc |

> 完整列表见 `base-ui-flutter/README.md`。

---
> Source: [SpringHgui/daro](https://github.com/SpringHgui/daro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-10 -->
