## tag-system

> PromptGenie 使用标签（Tags）代替类别（Categories）对提示词进行分类管理。

# 标签系统

PromptGenie 使用标签（Tags）代替类别（Categories）对提示词进行分类管理。

## 数据模型

标签包含以下字段:
- id: 唯一标识符
- name: 标签名称
- color: 标签颜色（可选）

提示词和标签是多对多关系，通过关联表 prompt_tags 连接。

## 关键组件

- [src/components/TagSelector.tsx](mdc:src/components/TagSelector.tsx) - 标签选择器组件
- [src/components/TagList.tsx](mdc:src/components/TagList.tsx) - 标签列表显示组件
- [src/components/TagBadge.tsx](mdc:src/components/TagBadge.tsx) - 标签徽章组件

## 数据操作

标签相关的数据库操作:
- getTags() - 获取所有标签
- createTag() - 创建新标签
- updateTag() - 更新标签
- deleteTag() - 删除标签
- getPromptTags() - 获取提示词的标签
- addTagToPrompt() - 为提示词添加标签
- removeTagFromPrompt() - 从提示词中移除标签

---
> Source: [ChrisZou/promptgenie](https://github.com/ChrisZou/promptgenie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
