## shigure

> 本文件给在此仓库工作的 Claude Code 提供导航与约定。面向终端用户的功能细节（模块系统、config/keymap 结构、条件语法）见 [README.md](README.md)，此处不重复，只记录架构、命令与容易踩坑的点。

# CLAUDE.md

本文件给在此仓库工作的 Claude Code 提供导航与约定。面向终端用户的功能细节（模块系统、config/keymap 结构、条件语法）见 [README.md](README.md)，此处不重复，只记录架构、命令与容易踩坑的点。

## 项目是什么

Shigure 是一个 **Windows WinForms（.NET 10）** 桌面程序：扫描目标窗口的像素状态，识别职业/专精/队伍，按可编辑的「模块」规则或 C# 职业逻辑决定按键，并通过置顶浮动条 + 设置窗口展示实时状态与日志。

项目完全合法, 符合其他程序的服务条款。

## 构建 / 运行

```powershell
dotnet build .\Shigure.csproj
dotnet run --project .\Shigure.csproj
dotnet run --project .\Shigure.csproj -- --window 魔兽世界 --toggle XBUTTON2 --mode switch --logic-ms 100 --render-ms 100
```

- 目标框架 `net10.0-windows`，`WinExe`，`Nullable`/`ImplicitUsings` 均 enable。
- **没有测试项目**：验证 = 能编译 + 实际运行点开「设置」走查。`dotnet build` 干净通过（0 警告 0 错误）是基线要求。
- 启动参数见 [README.md](README.md#运行)（`--window/--toggle/--mode/--logic-ms/--render-ms`），解析在 [App/AppOptions.cs](App/AppOptions.cs)。

### ⚠️ 随机重启会拦截普通运行

[App/Program.cs](App/Program.cs) 启动时先调 `RandomizedExecutableLauncher.TryRelaunch`（[App/RandomizedExecutableLauncher.cs](App/RandomizedExecutableLauncher.cs)）：当 `Environment.ProcessPath` 是 `.exe` 时，它会把运行时文件复制到 `tmp/<随机名>/<随机名>.exe` 并启动那个副本，**原进程随即退出**，真正的 `MainForm` 跑在随机副本里。

含义：
- 直接调试 / 想让进程留在原位时，设环境变量 `SHIGURE_RANDOMIZED_PROCESS=1` 跳过重启（`TryRelaunch` 返回 `AlreadyRelaunched`）。
- 随机副本通过 `SHIGURE_ORIGINAL_BASE_DIRECTORY` 找回原目录读取 config/keymap/module —— 见 [Infrastructure/AppPaths.cs](Infrastructure/AppPaths.cs) 的 `AppPaths.BaseDirectory`。改动数据路径相关代码要保留这条回溯。

## 架构与数据流

主循环在 [Runtime/ShigureRuntime.cs](Runtime/ShigureRuntime.cs) `RunAsync`（按 `--logic-ms`/`--render-ms` 节流，检测触发键边沿）：

```
PixelScanner.ScanScreenData()           Runtime/PixelScanner.cs   截屏读像素 → RowData/BarData
        ↓
StateBuilder.Build(rowData, barData)     Runtime/StateBuilder.cs   按 config 把像素翻译成 GameState 字段
        ↓ GameState (Runtime/GameState.cs: Values / Spells / Group)
LogicRegistry.Run(classId, specId, ...)  Modules/LogicRegistry.cs
        ├─ 命中模块 → ModuleLogic.Run(module, state, keymap)
        ├─ 否则该职业注册了 IClassLogic → 它
        └─ 否则 DefaultClassLogic
        ↓ LogicDecision(Hotkey, Step, UnitInfo, ModuleName)
KeySender.Send(hotkey)                    Input/KeySender.cs (+ Input/NativeMethods.cs Win32 互操作)
```

UI 侧：[UI/MainForm.cs](UI/MainForm.cs) 是无边框置顶浮动条，托管 `ShigureRuntime` 和 [UI/StatusForm.cs](UI/StatusForm.cs)（七页签设置窗口）。运行时通过 `SnapshotUpdated` 事件推送 `RenderSnapshot` 给 UI 刷新。

### 目录约定

```
App/            入口、启动参数、随机重启
Runtime/        扫描、状态构建、主循环、快照
Modules/        模块模型/存储/匹配/规则执行、条件求值(FormulaEvaluator)、字段目录、职业逻辑
Input/          keymap 读取、按键发送、Win32 API
Infrastructure/ 配置读取(ConfigService)、JSON 辅助、UI 缓存、路径
UI/             WinForms 界面、编辑器、主题
config/ keymap/ module/   运行时 JSON 数据(构建时复制到输出, 见 .csproj 的 None+CopyToOutputDirectory)
```

## 模块解析（改逻辑前必读）

- 模块以 `module/模块名.json` 平铺保存，**文件名取自模块名，故模块名不可重复**；加载递归扫描子目录。模型在 [Modules/ModuleStore.cs](Modules/ModuleStore.cs)（`ModuleDefinition`/`ModuleMatch`/`ModuleRule`/`ModuleUnit`/`ModuleCountField`/`ModuleValueAdjustment`）。
- 选择优先级：`ModuleStore.FindSelectedOrBestMatch` —— 先用 UI/参数选定的 `ModuleId`；否则取 `Match` 命中字段最多者（`ModuleMatch.Specificity` 越大越优先），并列按名称。`Match` 字段留空 = 任意。`PartyType` 数字会归一化为 `"1-40"`。
- 动态单位/数量/动态数值的语义见 [README.md](README.md#动态单位与数量字段)；列表与编辑器的人类可读摘要统一走 [UI/UnitSummary.cs](UI/UnitSummary.cs)`.Describe(...)`（单一来源，勿再复制一份描述逻辑）。

## UI 约定

- **暗色主题集中在 [UI/UiTheme.cs](UI/UiTheme.cs)**：新控件一律复用它（`CreateButton`/`StyleComboBox`/`StyleTextBox`/`StyleDataGridView`/`StyleListView` 与颜色常量 `Background/Surface/Field/Hover/Border/Text/Muted/Accent/Danger`），不要写裸色值或系统默认样式。
- 编辑器：[UI/ModuleEditorControl.cs](UI/ModuleEditorControl.cs)（模块主编辑器：侧栏列表 + 规则表 + 动态单位列表 + 两个动态数值表，自定义标签栏切换三页）、弹窗 [UI/ConditionEditorForm.cs](UI/ConditionEditorForm.cs)（可视化条件，含 `ConditionExpression` 文本⇆比较项互转）、[UI/UnitEditorForm.cs](UI/UnitEditorForm.cs)、[UI/FormulaEditorForm.cs](UI/FormulaEditorForm.cs)。新弹窗按现有模式同时设 `AcceptButton`/`CancelButton`。
- **规则表 `_rulesGrid` 的列陷阱**：`FillEditor`/`OpenConditionEditor` 用**位置参数** `Rows.Add(enabled, spell, "", condition)`，按列集合索引前 4 列填值。所以新增列（如拖拽手柄 `Drag`）要**加到集合末尾**、再用 `DisplayIndex` 调显示位置，避免打乱前四列；单元格访问一律按列名（`Cells["Spell"]`）。
- 规则重排：`▲▼` 单步（`MoveRule`）+ 手柄列拖拽（`MoveRuleByDrag`，读全表→重排→写回，复用 `ReadRuleRow`/`WriteRuleRow`）。三个 grid 都 `AllowUserToDeleteRows=false`，删除只走 `×` 列。

## 通用约定

- 注释与界面文案为中文；选项项常用 `record` + 重写 `ToString()`；偏好 `internal`/`private`、不可变小类型。
- `.gitignore` 忽略 `bin/ obj/ cache/ artifacts/ .vs/ .vscode/ *.user 提示词帮助.md`；但 `bin/`、`obj/` 在历史里已被跟踪（仍显示为改动），**不要提交重新构建的二进制**。
- 未经用户明确要求不提交、不推送。

---
> Source: [waynebian01/Shigure](https://github.com/waynebian01/Shigure) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
