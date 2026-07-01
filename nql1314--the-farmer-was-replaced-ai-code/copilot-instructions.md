## readme

> 本文件夹包含了 The Farmer Was Replaced 游戏开发的所有规则和参考文档。

# TFWR Cursor 规则索引

本文件夹包含了 The Farmer Was Replaced 游戏开发的所有规则和参考文档。

## 📚 规则文件列表

### 🔴 始终启用的规则（alwaysApply: true）

#### 1. [project-overview.mdc](mdc:project-overview.mdc)
- **用途**：项目概述和基本说明
- **内容**：项目结构、重要注意事项、游戏基础机制
- **何时查阅**：项目初始化、了解项目整体结构

#### 2. [coding-standards.mdc](mdc:coding-standards.mdc)
- **用途**：TFWR 编码规范
- **内容**：
  - 注释规范（禁止 `"""`, 只用 `#`）
  - 禁止使用的功能（lambda, yield, int() 等）
  - 字典使用限制（不支持 .items(), .keys(), .values()）
  - 代码风格指南
- **何时查阅**：编写新代码前、遇到语法错误

#### 3. [common-issues.mdc](mdc:common-issues.mdc)
- **用途**：常见问题和解决方案
- **内容**：
  - 语法限制（重要！）
  - 常见错误及解决方案
  - 解锁顺序建议
  - 资源管理策略
  - 特殊机制说明
  - 调试工具使用
  - 最佳实践
- **何时查阅**：遇到错误、规划解锁顺序、优化资源获取

#### 4. [game-mechanics.mdc](mdc:game-mechanics.mdc)
- **用途**：详细的游戏机制说明
- **内容**：
  - 巨型农场（多无人机系统）
  - 迷宫系统
  - 肥料和感染机制
  - 浇水系统
  - 向日葵和能量
  - 南瓜合并机制
  - 仙人掌排序机制
  - 混合种植系统
  - 恐龙帽系统
  - 调试和模拟工具
- **何时查阅**：使用高级功能、优化特定作物产量

#### 5. [quick-reference.mdc](mdc:quick-reference.mdc)
- **用途**：快速参考数据
- **内容**：
  - 作物数据（生长时间、成本、产量）
  - 操作成本（Tick 消耗）
  - 解锁顺序建议
  - 资源获取路径
  - 常用常量和函数模板
  - 性能优化清单
  - 常见错误速查表
- **何时查阅**：查询具体数据、快速查找函数模板

### 🟢 特定文件类型规则（globs: *.py）

#### 6. [code-patterns.mdc](mdc:code-patterns.mdc)
- **Globs**: `*.py`
- **用途**：常见代码模式和算法
- **内容**：
  - 网格遍历模式（蛇形、行列）
  - 植物管理模式
  - 物品管理
  - 特定作物优化
  - 调试技巧
- **何时查阅**：需要实现遍历算法、管理作物

#### 7. [game-api-usage.mdc](mdc:game-api-usage.mdc)
- **Globs**: `*.py`
- **用途**：游戏 API 使用规则和最佳实践
- **内容**：
  - 核心 API 导入（Entities, Items, Directions, Grounds）
  - 常用函数模式（移动、收割、种植）
  - Tick 优化建议
- **何时查阅**：使用游戏 API、优化操作成本

#### 8. [optimization-tips.mdc](mdc:optimization-tips.mdc)
- **Globs**: `*.py`
- **用途**：性能优化技巧和高级策略
- **内容**：
  - Tick 成本概览
  - 优化策略（移动、缓存、批处理等）
  - 高级技巧（能量、并行、测量）
  - 算法复杂度考虑
  - 调试和性能分析
- **何时查阅**：优化代码性能、减少 Tick 消耗

## 🎯 使用指南

### 开始新项目时
1. 阅读 [project-overview.mdc](mdc:project-overview.mdc) 了解项目结构
2. 阅读 [coding-standards.mdc](mdc:coding-standards.mdc) 了解编码规范
3. 查看 [quick-reference.mdc](mdc:quick-reference.mdc) 了解基础数据

### 编写代码时
1. 参考 [code-patterns.mdc](mdc:code-patterns.mdc) 使用标准模式
2. 查询 [game-api-usage.mdc](mdc:game-api-usage.mdc) 使用正确的 API
3. 应用 [optimization-tips.mdc](mdc:optimization-tips.mdc) 优化性能

### 遇到问题时
1. 查看 [common-issues.mdc](mdc:common-issues.mdc) 找常见问题
2. 查询 [quick-reference.mdc](mdc:quick-reference.mdc) 的错误速查表
3. 检查 [coding-standards.mdc](mdc:coding-standards.mdc) 确认语法正确

### 使用高级功能时
1. 阅读 [game-mechanics.mdc](mdc:game-mechanics.mdc) 了解详细机制
2. 参考 [quick-reference.mdc](mdc:quick-reference.mdc) 查询数据
3. 应用 [optimization-tips.mdc](mdc:optimization-tips.mdc) 优化

## 📊 规则优先级

当规则之间有冲突时，遵循以下优先级：

1. **编码规范** (coding-standards.mdc) - 最高优先级
   - 语法限制必须遵守
   
2. **游戏机制** (game-mechanics.mdc, common-issues.mdc)
   - 游戏规则不可违反
   
3. **优化建议** (optimization-tips.mdc)
   - 在符合规范的前提下优化
   
4. **代码模式** (code-patterns.mdc)
   - 参考但不强制

## 🔍 快速查找

### 按主题查找

#### 语法和规范
- 注释规范 → [coding-standards.mdc](mdc:coding-standards.mdc)
- 禁止使用的功能 → [coding-standards.mdc](mdc:coding-standards.mdc)
- 字典使用 → [coding-standards.mdc](mdc:coding-standards.mdc)

#### 游戏机制
- 多无人机 → [game-mechanics.mdc](mdc:game-mechanics.mdc) § 巨型农场
- 迷宫 → [game-mechanics.mdc](mdc:game-mechanics.mdc) § 迷宫
- 肥料 → [game-mechanics.mdc](mdc:game-mechanics.mdc) § 肥料系统
- 浇水 → [game-mechanics.mdc](mdc:game-mechanics.mdc) § 浇水系统
- 南瓜 → [game-mechanics.mdc](mdc:game-mechanics.mdc) § 南瓜系统
- 仙人掌 → [game-mechanics.mdc](mdc:game-mechanics.mdc) § 仙人掌系统
- 向日葵 → [game-mechanics.mdc](mdc:game-mechanics.mdc) § 向日葵系统

#### 数据查询
- 作物数据 → [quick-reference.mdc](mdc:quick-reference.mdc) § 作物数据
- Tick 成本 → [quick-reference.mdc](mdc:quick-reference.mdc) § 操作成本
- 解锁顺序 → [quick-reference.mdc](mdc:quick-reference.mdc) § 解锁顺序建议
- 资源获取 → [quick-reference.mdc](mdc:quick-reference.mdc) § 资源获取路径

#### 代码模式
- 遍历模式 → [code-patterns.mdc](mdc:code-patterns.mdc) § 网格遍历模式
- 植物管理 → [code-patterns.mdc](mdc:code-patterns.mdc) § 植物管理模式
- 函数模板 → [quick-reference.mdc](mdc:quick-reference.mdc) § 实用函数模板

#### 优化技巧
- Tick 优化 → [optimization-tips.mdc](mdc:optimization-tips.mdc)
- 移动优化 → [optimization-tips.mdc](mdc:optimization-tips.mdc) § 优化策略
- 性能分析 → [optimization-tips.mdc](mdc:optimization-tips.mdc) § 调试和性能分析

## 📝 更新日志

### 最新更新
- ✅ 创建详细的游戏机制文档 (game-mechanics.mdc)
- ✅ 创建快速参考文档 (quick-reference.mdc)
- ✅ 修复 coding-standards.mdc 中的重复内容
- ✅ 创建规则索引文档 (README.mdc)

### 规则覆盖范围
- ✅ 编码规范
- ✅ 常见问题
- ✅ 游戏机制（全部高级功能）
- ✅ 性能优化
- ✅ 代码模式
- ✅ API 使用
- ✅ 快速参考

## 💡 提示

- 所有规则都设置为 `alwaysApply: true`，Cursor AI 会自动引用
- 使用 Ctrl+P 搜索文件名快速访问
- 规则文件使用 `.mdc` 格式，支持 Markdown 语法
- 建议在编码前快速浏览相关规则
- 遇到问题先查规则，再搜索，最后才问 AI

## 🔗 相关资源

- 游戏 API 定义：[__builtins__.py](mdc:../../__builtins__.py)
- 项目根目录：`C:\Users\49479\AppData\LocalLow\TheFarmerWasReplaced\TheFarmerWasReplaced\Saves\cursor`
- 游戏存档：[save.json](mdc:../../save.json)

---
> Source: [nql1314/The-Farmer-Was-Replaced-AI-Code](https://github.com/nql1314/The-Farmer-Was-Replaced-AI-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-01 -->
