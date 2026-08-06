## xiaofutools

> - .NET 10、WPF、ArcGIS Pro SDK 3.7+、Windows x64。

# XIAOFUTools 代理开发规范

## 项目基线

- .NET 10、WPF、ArcGIS Pro SDK 3.7+、Windows x64。
- 单一生产工程 `XIAOFUTools.csproj`，测试工程位于 `tests/XIAOFUTools.Tests`。
- 完整架构见 `docs/architecture.md`，开发流程见 `docs/development.md`。

## 必须遵守

- 保持现有 DAML ID、DockPane ID、设置 JSON、SQLite 模式和输出格式兼容。
- ArcGIS Core/Mapping API 在 `QueuedTask` 或 `IArcGisTaskRunner` 中运行。
- 新代码按功能放入 `Features/<Domain>/<Feature>`；只有跨功能复用代码进入 `Shared`。
- `Shared` 禁止引用 `Features`，功能之间禁止引用对方的 ViewModel 和窗口。
- 复用 `Shared/Mvvm` 命令、状态、日志和取消机制，不得新增私有 `RelayCommand`。
- ViewModel 不直接调用 MessageBox、文件对话框、剪贴板或外部进程。
- WPF 控件使用 `Shared/Presentation/Styles/ControlStyles.xaml` 的统一样式。
- 保护现有未提交改动，不得使用 `git reset --hard` 或覆盖用户文件。

## 功能结构

小型功能保持扁平结构：

```text
Feature/
├── FeatureButton.cs
├── FeatureDockPane.cs
├── FeatureDockPaneView.xaml
├── FeatureDockPaneView.xaml.cs
└── FeatureViewModel.cs
```

包含复杂规则、数据库、网络、Office 或多个界面的功能采用：

```text
Feature/
├── Core/
├── Application/
├── Infrastructure/
└── Presentation/
```

## 完成条件

- Debug/Release x64 构建成功并生成 `.esriAddinX`。
- 默认测试使用 `Category!=LiveNetwork` 全部通过。
- DAML 类名、内部引用、图标和资源架构测试通过。
- 新增功能包含错误处理、取消、进度、日志、帮助和对应测试。

---
> Source: [xiaofuX1/XIAOFUTools](https://github.com/xiaofuX1/XIAOFUTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
