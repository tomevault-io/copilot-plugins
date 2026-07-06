## ue-cpp-learn

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 项目概述

这是一套**从零开始的教学手册**，包含17个章节、共75个中文`.md`文件。纯文档项目，无构建系统、测试或可运行代码。目标受众为零基础、学习虚幻引擎5 C++开发的初学者。

## 内容规范

编写或编辑本项目中的任何`.md`文件时，请遵循以下规则：

- **语言**：所有解释说明必须使用中文（简体中文）。代码标识符和UE API名称保留英文。
- **代码注释**：每行C++代码必须附带行内中文注释，说明其作用及原因。
- **正确/错误标记**：正确示例使用`✅`，常见错误使用`❌`。必须同时展示正反两面。
- **傻瓜式引导**：假定读者从未做过这件事。需包含编辑器菜单路径、UI按钮名称和预期结果。
- **ASCII图示**：对于架构、数据流和内存布局的解释，使用基于文本的图表。
- **对比表格**：对比相似概念时优先使用表格（例如：FString vs FName vs FText、BlueprintCallable vs BlueprintNativeEvent）。
- **章节末尾检查清单**：每个主要章节结束时均设有`- [ ]`检查清单，供读者检验理解。
- **章节案例**：每个章节末尾附带一个综合性动手案例（`XX-章节案例.md`），串联该章节所有概念。

## 章节结构

每个章节遵循以下模式：

```
XX-章节名/
├── 01-知识点1.md       ← 核心教学内容
├── 02-知识点2.md       ← 核心教学内容
├── ...
├── YY-章节案例.md      ← 综合性案例，带完整注释代码
└── 案例源码/           ← 案例源码（如有）
```

章节按渐进式学习顺序排列：环境搭建 → C++基础 → UE核心概念 → 中级系统 → GAS → 优化 → 真实项目 → 求职准备。

## 示例中的UE5代码风格

所有代码示例必须遵循UE5规范和最佳实践：

- 类前缀：`A`（Actor）、`U`（UObject）、`F`（结构体）、`I`（接口）、`E`（枚举）、`T`（模板）
- 每个UE类中必须包含`UCLASS()`、`UPROPERTY()`、`UFUNCTION()`、`GENERATED_BODY()`
- `.generated.h`作为最后一个包含头文件
- 布尔变量以`b`开头（例如`bIsAlive`）
- 优先使用UE类型而非标准C++类型：使用`int32`而非`int`、`TArray`而非`std::vector`、`FString`而非`std::string`
- 所有字符串字面量使用`TEXT()`宏包裹
- 使用`Super::`调用父类方法

---
> Source: [DaNiuNai/ue-cpp-learn](https://github.com/DaNiuNai/ue-cpp-learn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
