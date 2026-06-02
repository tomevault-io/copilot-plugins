## yelihu-github-io

> **Ye Lihu 的个人技术博客** - 使用 Hexo + Icarus 主题构建

# CLAUDE.md

**Ye Lihu 的个人技术博客** - 使用 Hexo + Icarus 主题构建

## 项目概述

这是我的个人技术博客，使用 Hexo 静态网站生成器和 Icarus 主题，专注于分享技术文章和学习心得。

## 常用命令

### 开发
- `hexo server` - 启动本地开发服务器（默认端口：4000）
- `hexo generate` - 生成静态文件
- `hexo clean` - 清理缓存

### 内容管理
- `hexo new <title>` - 创建新博客文章
- `hexo new page <name>` - 创建新页面

### 部署
- `git push origin main` - 自动触发 GitHub Actions 部署

## 配置文件

- `_config.yml` - Hexo 主配置
- `_config.icarus.yml` - Icarus 主题配置
- `source/_posts/` - 博客文章目录
- `source/img/` - 图片资源目录

## 部署方式

使用 GitHub Actions 自动部署到 GitHub Pages，推送代码即可自动发布。

## 文章规范

### Front-matter 模板

**基础模板：**
```yaml
---
title: 文章标题
date: 2025-01-01 12:00:00
author: Ye Lihu
toc: true
categories:
  - 分类名
tags:
  - 标签名
---
```

**📖 详细说明**：完整的 Front-matter 属性说明、模板示例和最佳实践请参考 @FRONTMATTER.md

### 快速参考
- **单级分类**：`categories: - AI`
- **多级分类**：`categories: - Java - 分布式中间件`
- **标签设置**：多个标签用数组形式

## 🚨 重要限制

**当用户要求创建新文章时，必须遵守以下规则：**

1. **只执行命令**：仅运行 `hexo new <title>` 创建文章文件
2. **只设置Front-matter**：仅配置文章头部的元数据信息
3. **严禁写正文**：绝对不能帮用户编写文章内容，包括前言、正文、总结等任何实质性内容
4. **提供指导**：可以告知用户如何开始写作，但不代写内容

**正确做法示例：**
- ✅ 执行 `hexo new "文章标题"`
- ✅ 配置 Front-matter（title、date、author、categories、tags等）
- ✅ 告知用户："文章已创建，请开始编写正文内容"

**错误做法示例：**
- ❌ 编写文章前言、正文段落
- ❌ 提供详细的代码示例和解释
- ❌ 撰写总结或结论部分

**原则：** 只提供脚手架，不提供内容创作。

---
> Source: [yelihu/yelihu.github.io](https://github.com/yelihu/yelihu.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
