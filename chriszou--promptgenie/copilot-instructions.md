## prompt-management

> 应用的核心功能是提示词（Prompt）的管理，包括添加、编辑、删除和标记收藏。

# 提示词管理

应用的核心功能是提示词（Prompt）的管理，包括添加、编辑、删除和标记收藏。

## 数据模型

提示词包含以下字段:
- id: 唯一标识符
- title: 标题
- content: 提示词内容
- tags: 标签列表（用于分类和筛选）
- isFavorite: 是否收藏
- createdAt: 创建时间
- updatedAt: 更新时间

## 关键组件

- [src/components/PromptList.tsx](mdc:src/components/PromptList.tsx) - 提示词列表显示组件
- [src/components/PromptCard.tsx](mdc:src/components/PromptCard.tsx) - 提示词卡片组件
- [src/components/PromptForm.tsx](mdc:src/components/PromptForm.tsx) - 提示词编辑表单

## 数据操作

所有数据库操作通过 [src/services/db.ts](mdc:src/services/db.ts) 中的服务函数实现:
- getPrompts() - 获取所有提示词
- getPromptById() - 根据ID获取提示词
- createPrompt() - 创建新提示词
- updatePrompt() - 更新提示词
- deletePrompt() - 删除提示词
- toggleFavorite() - 切换收藏状态

---
> Source: [ChrisZou/promptgenie](https://github.com/ChrisZou/promptgenie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
