## target

> 学习 Midscene 基础概念

# Midscene 核心代码学习指南

## 仓库概述

这是一个用于学习 Midscene 核心代码的仓库。Midscene 是一个基于 AI 的浏览器自动化框架，支持使用自然语言描述来执行浏览器操作、查询页面内容、进行断言等。

## 核心概念

### 1. Agent（代理）
- **核心类**: `Agent` 来自 `@midscene/core`
- **作用**: Agent 是 Midscene 的核心抽象，负责与设备（浏览器、移动设备等）交互
- **关键方法**:
  - `aiAction(prompt)`: 执行 AI 驱动的操作（点击、输入、滚动等）
  - `aiQuery(prompt)`: 查询页面内容并返回结构化数据
  - `aiWaitFor(prompt)`: 等待某个条件满足
  - `aiAssert(prompt)`: 使用 AI 进行断言
  - `aiTap(prompt)`: 点击元素
  - `aiBoolean(prompt)`: 返回布尔值
  - `aiNumber(prompt)`: 返回数字
  - `aiString(prompt)`: 返回字符串
  - `aiLocate(prompt)`: 定位元素位置

### 2. 平台实现
- **PlaywrightAgent**: `@midscene/web/playwright` - 基于 Playwright 的实现
- **PuppeteerAgent**: `@midscene/web/puppeteer` - 基于 Puppeteer 的实现
- **Android/iOS Agent**: 支持移动端自动化

### 3. 设备接口（Device Interface）
- 支持自定义设备实现
- 参考 `custom-interface/src/sample-device.ts` 了解如何实现自定义设备
- 设备需要实现特定的接口方法以支持 Agent 的操作

### 4. YAML 脚本支持
- 支持使用 YAML 文件定义自动化流程
- 参考 `yaml-scripts-demo/midscene-scripts/` 中的示例
- 主要配置项：
  - `web`: 浏览器配置（URL、viewport、输出等）
  - `tasks`: 任务列表，每个任务包含 `flow` 步骤

## 学习路径

### 阶段 1: 理解基本使用
1. **查看示例代码**:
   - `playwright-demo/demo.ts` - Playwright 基础用法
   - `puppeteer-demo/demo.ts` - Puppeteer 基础用法
   - `yaml-scripts-demo/midscene-scripts/search-headphone-on-ebay.yaml` - YAML 脚本示例

2. **关键学习点**:
   - Agent 的初始化方式
   - 各种 AI 方法的使用场景
   - 如何结合现有的浏览器自动化框架

### 阶段 2: 深入核心架构
1. **核心模块**:
   - `@midscene/core` - 核心 Agent 类和抽象
   - `@midscene/web/playwright` - Playwright 适配器
   - `@midscene/web/puppeteer` - Puppeteer 适配器

2. **需要理解的问题**:
   - Agent 如何与 LLM 交互？
   - 如何将自然语言转换为浏览器操作？
   - 截图和页面分析如何工作？
   - 缓存机制如何实现？

### 阶段 3: 自定义扩展
1. **自定义设备**:
   - 参考 `custom-interface/src/` 目录
   - 了解如何实现 `Device` 接口
   - 如何创建自定义 Agent

2. **集成测试**:
   - `playwright-testing-demo/` - Playwright 测试集成
   - `puppeteer-with-vitest-demo/` - Puppeteer + Vitest 集成

## 关键文件结构

```
midscene-example/
├── playwright-demo/          # Playwright 集成示例
├── puppeteer-demo/           # Puppeteer 集成示例
├── yaml-scripts-demo/        # YAML 脚本示例
├── custom-interface/         # 自定义设备接口示例
├── bridge-mode-demo/         # Bridge 模式（桌面 Chrome）
├── android/                  # Android 平台示例
├── ios/                      # iOS 平台示例
└── connectivity-test/        # LLM 连接测试
```

## 学习重点

### 1. Agent 的工作流程
- 接收自然语言指令
- 分析当前页面状态（截图、DOM）
- 调用 LLM 理解指令并生成操作
- 执行操作并验证结果

### 2. 与 LLM 的交互
- 需要配置 LLM API（通过环境变量）
- 了解 prompt 工程如何优化指令理解
- 错误处理和重试机制

### 3. 页面理解机制
- 截图分析
- DOM 结构分析
- 视觉元素定位

### 4. 缓存机制
- 操作结果的缓存
- 页面状态的缓存
- 如何利用缓存提高效率

## 调试技巧

1. **查看示例代码**: 每个 demo 目录都有完整的示例
2. **运行示例**: 使用 `npm run test` 或 `tsx demo.ts` 运行示例
3. **查看 YAML 脚本**: 理解 YAML 格式如何转换为代码执行
4. **自定义设备**: 通过实现自定义设备理解核心接口

## 常见问题

1. **如何配置 LLM API?**
   - 使用 `.env` 文件配置环境变量
   - 参考 `connectivity-test/` 测试连接

2. **如何自定义 Agent?**
   - 继承 `Agent` 类
   - 参考 `custom-interface/src/index.ts` 中的 `SampleDeviceAgent`

3. **如何集成到测试框架?**
   - 参考 `playwright-testing-demo/` 和 `puppeteer-with-vitest-demo/`

## 学习目标

通过这个仓库，你应该能够：
1. ✅ 理解 Midscene 的核心架构和设计理念
2. ✅ 掌握 Agent 的使用方法和各种 AI 操作
3. ✅ 了解如何在不同平台（Web、Android、iOS）使用 Midscene
4. ✅ 学会自定义设备接口和扩展 Agent
5. ✅ 理解 Midscene 如何与 LLM 交互实现智能自动化

## 代码分析建议

当分析 Midscene 核心代码时，重点关注：
- **抽象层设计**: Agent 如何抽象不同平台的差异
- **LLM 集成**: 如何将自然语言转换为可执行操作
- **错误处理**: 如何处理操作失败和重试
- **性能优化**: 缓存机制和批量操作
- **扩展性**: 如何支持新的平台和设备类型

---
> Source: [wangjs-jacky/jacky-midscene](https://github.com/wangjs-jacky/jacky-midscene) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
