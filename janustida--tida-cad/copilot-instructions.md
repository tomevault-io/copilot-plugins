## tida-cad

> 本文件为 AI 编码代理（Codex (Codex.ai/code)、Claude Code 等）在此代码库中工作时提供指导。

# AGENTS.md

本文件为 AI 编码代理（Codex (Codex.ai/code)、Claude Code 等）在此代码库中工作时提供指导。

## 项目概述

Tida.CAD 是一个基于 .NET 的 CAD 画布框架，提供平台无关的核心层（图层、绘制对象、坐标转换、输入与事件），并内置 **WPF** 与 **Avalonia** 两套 UI 实现（同一套绘制代码可运行于两个框架）。核心为共享工程 `Tida.CAD/`，通过条件编译（`#if WPF` / `#if Avalonia`）区分平台代码；渲染采用每图层一个 `DrawingVisual`，绘制对象内容以 `DrawingGroup` 缓存。

## 工具使用

- 没有特殊要求的情况下，优先使用内置的 Edit 工具，而不是生成 PowerShell 或 Python 脚本来做临时的编辑操作
- 如果一定要使用脚本修改文件，在Windows 环境下修改文件内容优先使用 **PowerShell**（.ps1 脚本），避免使用 Python 处理文件读写，防止字符串转义问题
- 脚本先写入 `c:\tmp\` 再通过 `powershell.exe -ExecutionPolicy Bypass -File` 执行

## 命名规范

### C#
- 类名：`PascalCase` (`CADLayer`)
- 接口：`I` + `PascalCase` (`ICADControl`)
- 方法：`PascalCase`，异步方法以 `Async` 结尾 (`GetAllAsync()`)
- 私有字段：`_camelCase` (`_layerVisualDict`)

### XAML
- 控件：前缀 + `PascalCase` (`btnAdd`, `txtCategoryName`, `dgCategories`)

## 代码标准

### 异步编程
- 异步方法必须以 `Async` 结尾
- 使用 `Task` 或 `Task<T>` 返回类型

### 控制流语句
- 所有条件语句（`if`/`else if`/`else`）和循环语句（`for`/`foreach`/`while`/`do-while`），无论后续语句数量是一条还是多条，都必须使用花括号 `{}`

### 方法调用换行格式
- 新生成的代码中，方法调用行过长时，左括号 `(` 移到新行：
```csharp
// 错误
var config = _service.GetConfig(
    param1, param2);

// 正确
var config = _service.GetConfig
(
    param1,
    param2
);
```

### LINQ 风格
- 优先使用 LINQ 拓展方法（方法语法）而非查询表达式（类 SQL 语法）
- 正确：`list.Where(x => x > 0).Select(x => x.ToString())`
- 错误：`from x in list where x > 0 select x.ToString()`

### 嵌套深度
- 新添加的代码应尽量减少嵌套层级（建议不超过 3 层）
- 优先使用提前返回来减少 `if` 嵌套（如 `if (!condition) return;`）
	- 循环中使用 `if (!condition) continue;` 替代 `if (condition) { ... }`，减少一层缩进
- 复杂逻辑块应提取为独立的私有方法，以降低主方法的圈复杂度

## 附加说明

- 代码注释和日志消息使用中文
- 代码文件尽量都保存成utf-8 with bom编码

---
> Source: [JanusTida/Tida.CAD](https://github.com/JanusTida/Tida.CAD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
