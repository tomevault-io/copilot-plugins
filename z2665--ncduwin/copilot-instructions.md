## ncduwin

> ncduwin 是一款基于 .NET 10 和 .NET MAUI (Blazor Hybrid) 的 Windows 磁盘使用分析工具，v1.7 版本采用现代扁平化设计风格。

# ncduwin - Agent 开发指南

## 项目概述

ncduwin 是一款基于 .NET 10 和 .NET MAUI (Blazor Hybrid) 的 Windows 磁盘使用分析工具，v1.7 版本采用现代扁平化设计风格。

## 代码结构

```
ncduwin/
├── NcduWin.Core/              # 核心类库 (无UI依赖)
│   ├── Models/                # 数据模型
│   │   ├── FileSystemItem.cs  # 文件系统项基类
│   │   ├── DirectoryItem.cs   # 目录项
│   │   ├── FileItem.cs        # 文件项
│   │   ├── ScanProgress.cs    # 扫描进度
│   │   ├── ScanOptions.cs     # 扫描选项
│   │   ├── AppSettings.cs     # 应用配置
│   │   ├── SnapshotInfo.cs    # 快照元数据
│   │   ├── SnapshotData.cs    # 快照数据
│   │   └── FlatFileSystemItem.cs # 扁平化文件项
│   ├── Interfaces/            # 接口定义
│   │   ├── IDiskScanner.cs    # 磁盘扫描器接口
│   │   ├── ISettingsService.cs
│   │   ├── IFileIconService.cs
│   │   └── ISnapshotService.cs # 快照服务接口
│   └── Services/              # 服务实现
│       ├── DiskScanner.cs     # 核心扫描逻辑
│       ├── SettingsService.cs # 配置管理
│       ├── FileIconService.cs # 图标映射服务
│       └── SnapshotService.cs # 快照管理服务
│
├── ncduwin/                   # MAUI Blazor UI 层
│   ├── Components/
│   │   ├── Pages/
│   │   │   ├── Home.razor     # 主页面 (业务逻辑入口)
│   │   │   └── NotFound.razor
│   │   ├── Ncdu/              # 功能组件
│   │   │   ├── FileListView.razor    # 文件列表视图
│   │   │   ├── ScanDashboard.razor   # 扫描面板
│   │   │   ├── Breadcrumbs.razor     # 面包屑导航
│   │   │   ├── StatusBar.razor       # 状态栏
│   │   │   ├── ScanOverlay.razor     # 扫描浮层
│   │   │   ├── ActionsMenu.razor      # 右键菜单
│   │   │   ├── DeleteConfirmDialog.razor
│   │   │   ├── PropertiesDialog.razor
│   │   │   ├── SettingsDialog.razor
│   │   │   ├── AboutDialog.razor
│   │   │   └── SnapshotSidebar.razor  # 快照侧边栏
│   │   └── Layout/
│   │       └── MainLayout.razor
│   ├── Resources/AppIcon/     # 应用图标
│   ├── wwwroot/
│   │   ├── icons/            # SVG 文件图标库 (~100种文件类型)
│   │   ├── app.css           # 全局样式 (CSS变量系统)
│   │   └── index.html
│   ├── MauiProgram.cs         # MAUI 启动配置
│   └── ncduwin.csproj
│
├── NcduWin.Core.Tests/        # 单元测试 (xUnit)
├── NcduWin.Core.Benchmarks/   # 性能基准测试 (BenchmarkDotNet)
├── spec/                      # 规格文档
│   ├── spec.txt               # 主规格文档
│   └── enhance.txt            # 增强计划
└── docs/plans/                # 设计文档
```

## 开发规则

### 1. 架构原则

- **Core 层独立**: `NcduWin.Core` 不依赖任何 UI 框架
- **依赖注入**: 所有服务通过 DI 注入，在 `MauiProgram.cs` 注册
- **异步优先**: 文件系统操作必须使用 `async/await`

### 2. 代码规范

```csharp
// 服务接口放在 Interfaces/，实现在 Services/
public interface IService { }
public class Service : IService { }

// 模型放在 Models/，使用 PascalCase
public class MyModel
{
    public string Name { get; set; }
}

// Blazor 组件: 一个 .razor 文件包含 HTML + CSS + C#
// 不添加注释，代码自解释
```

### 3. 组件开发

- 新组件放在 `Components/Ncdu/`
- 使用 `[Parameter]` 接收参数
- 使用 `EventCallback` 向父组件传递事件
- 样式写在组件内 `<style>` 块
- 全局样式变量定义在 `wwwroot/app.css`

### 4. 样式系统 (v1.7+)

使用 CSS 变量实现统一的主题：

```css
:root {
    --color-primary: #0F172A;      /* 深蓝主色 */
    --color-cta: #0369A1;          /* CTA 蓝色 */
    --color-background: #F8FAFC;   /* 背景色 */
    --color-surface: #FFFFFF;      /* 卡片背景 */
    --color-border: #E2E8F0;       /* 边框色 */
    --spacing-sm: 8px;
    --spacing-md: 16px;
    --radius-md: 8px;
    --transition-fast: 150ms ease;
}
```

### 5. 错误处理

- 扫描错误: 静默跳过，记录到日志
- UI 错误: 显示用户友好提示
- 使用 Serilog 记录日志: `Log.Information()`, `Log.Warning()`, `Log.Error()`

### 6. 状态管理

- 扫描完成后调用 `StateHasChanged()` 更新 UI
- Progress 回调使用 `InvokeAsync(StateHasChanged)` 确保线程安全

## 构建命令

```bash
# 构建 Windows 版本 (推荐)
dotnet build ncduwin/ncduwin.csproj -f net10.0-windows10.0.19041.0 -c Release

# 构建 Core 类库
dotnet build NcduWin.Core/NcduWin.Core.csproj

# 运行测试
dotnet test NcduWin.Core.Tests/

# 运行性能测试
dotnet run --project NcduWin.Core.Benchmarks/ -c Release
```

## 配置文件

| 文件 | 用途 |
|------|------|
| `settings.json` | 用户配置 (忽略目录列表)，位于 exe 同目录 |
| `logs/ncduwin.log` | Serilog 日志文件 |
| `snapshots/*.snapshot.gz` | 扫描结果快照文件 (JSON+Gzip) |

## 版本号

- UI 版本: `ncduwin/ncduwin.csproj` → `ApplicationDisplayVersion` (当前 v1.7)
- Core 版本: `NcduWin.Core/NcduWin.Core.csproj` → `Version`

## 已实现功能

- 目录/驱动器扫描 (支持取消、忽略规则)
- 文件列表展示 (虚拟化、排序、面包屑导航)
- 键盘操作 + 右键上下文菜单
- 文件操作 (打开、删除、复制路径、属性查看)
- About 窗口 (Core + UI 双版本显示)
- 配置管理 (忽略目录列表)
- 文件图标映射 (100+ SVG 图标)
- 错误处理与日志 (Serilog + 错误计数)
- 历史快照 (保存/加载 JSON+Gzip)
- 扁平化 UI 设计 (蓝色系、Poppins+Open Sans 字体)

## 注意事项

1. **图标文件必须命名为 `appicon.svg`** - MAUI Android 构建要求
2. **不要在 Core 层引用 MAUI/Blazor** - 保持独立性
3. **测试 Windows 目标** - 项目主要面向 Windows 用户
4. **更新 spec.txt** - 修改功能后更新规格文档
5. **完成功能开发后，请不要直接提交 commit，任何功能都需要经过审查后，由人手动提交**

---
> Source: [z2665/ncduwin](https://github.com/z2665/ncduwin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
