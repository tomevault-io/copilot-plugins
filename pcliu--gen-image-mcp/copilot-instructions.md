## gen-image-mcp

> - [src/tools/templates.ts](mdc:src/tools/templates.ts) - 模板管理工具

# 模板系统指南

## 模板管理

模板系统通过以下文件实现:
- [src/tools/templates.ts](mdc:src/tools/templates.ts) - 模板管理工具
- [src/services/template/index.ts](mdc:src/services/template/index.ts) - 模板服务
- [src/services/template/loader.ts](mdc:src/services/template/loader.ts) - 模板加载器

## 模板类型

系统支持三种基本模板类型:
1. illustration - 插图模板
2. concept - 概念图模板
3. diagram - 图表模板

## 模板结构

每个模板包含以下主要部分:
- 基本信息 (ID、名称、描述、版本)
- 样式定义 (艺术风格、配色、构图)
- 元素配置 (主要元素、辅助元素、文字样式)

## 模板应用流程

1. 加载基础模板
2. 应用覆盖参数
3. 生成完整提示词
4. 调用 AI 接口

## 开发指南

新增模板时需要:
1. 在 [src/services/template/index.ts](mdc:src/services/template/index.ts) 注册模板
2. 确保模板结构符合类型定义 [src/tools/types.ts](mdc:src/tools/types.ts)
3. 实现必要的验证和转换逻辑

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pcliu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
