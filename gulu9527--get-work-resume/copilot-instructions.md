## get-work-resume

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个个人简历生成系统，包含：

- 简历模板和生成工具
- Claude Code Skill (`/get-work-resume`) 用于根据 JD 生成定制化简历
- 模拟候选人数据用于测试

## 常用命令

无构建命令，纯静态 HTML 项目。

## 项目结构

```
.claude/skills/get-work-resume-skill/
├── SKILL.md          # 简历生成技能定义
└── template.html     # 简历模板

personal.json          # 用户个人信息数据
output/               # 生成的简历 HTML 文件
resume.html          # 原始简历模板
```

## 简历生成系统

### 使用 Skill

调用 `/get-work-resume` 并提供 JD（职位描述），系统会：

1. 读取 `personal.json` 获取个人信息
2. 分析 JD 提取技术栈要求
3. 根据 JD 优化简历内容
4. 生成 `output/日期-岗位-关键词.html`

### 模板变量

模板使用 Handlebars 语法，关键变量：

- `{{name}}` - 姓名
- `{{title}}` - 求职标题
- `{{techTags}}` - 技术栈标签数组
- `{{projects}}` - 项目经历数组
- `{{showPhoto}}` - 是否显示照片

### 修改模板

编辑 `.claude/skills/get-work-resume-skill/template.html`，然后同步更新 `output/` 中的示例文件。

### 添加新数据

编辑 `personal.json` 更新个人信息。字段说明：

- `photo` - 设为 `null` 显示灰色占位符
- `techTags` - 技术栈标签数组
- `projects` - 项目经历（包含 implementation 双列布局）

## 简历预览

直接在浏览器打开 `output/` 中的 HTML 文件：

- A4 尺寸 (210mm × 297mm)
- 右下角打印按钮
- 照片默认显示 1 寸灰色占位符

## 注意事项

- 照片尺寸：95px × 132px（约 1 寸）
- 模板改动后需同步到 output 目录示例文件
- 文件命名规则：`YYYY-MM-DD-岗位-关键词.html`

---
> Source: [GuLu9527/get-work-resume](https://github.com/GuLu9527/get-work-resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
