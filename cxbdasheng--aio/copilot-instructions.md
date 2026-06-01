## aio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

陈大剩的个人技术博客，基于 **MkDocs + Material 主题**，内容为家庭 AIO 服务器搭建教程（ESXi 虚拟化、软路由、黑群晖、远程穿透等）。在线地址：[aio.it927.com](https://aio.it927.com)

## 常用命令

### 本地开发（推荐用 Docker）

```bash
# 启动开发服务器（热重载，访问 http://localhost:8000）
docker compose up dev

# 或直接用 Python 本地运行（需先安装依赖）
pip install -r requirements.txt
mkdocs serve

# 构建静态文件
mkdocs build
```

### 生产部署

```bash
# 构建并启动生产容器（nginx，端口 80）
docker compose up prod
```

## 导航注册

**新增文章必须在 `mkdocs.yml` 的 `nav` 中注册**，否则文章不会出现在导航中。`nav` 按模块分组，对应 `docs/` 下的目录结构。

## 文章规范（摘要）

详细规则见 `.claude/rules/project_rules.md`，关键点：

- 语言：简体中文，技术术语保留英文
- Front Matter：需包含 `slug`、`keywords`、`description`（无 `title` 字段）
- 图片：全部使用 CDN（`https://img.it927.com/aio/<文件名>`），不在本地存储图片
- 代码块必须标注语言类型
- 文件命名：英文小写 + 连字符，如 `install-esxi.md`

## Git 提交规范

格式：`type(scope): 中文描述`

- `type`：`docs`（文章）、`chore`（配置/脚本）、`fix`（修正）
- `scope`：模块名，如 `remote`、`esxi`、`synology`
- 示例：`docs(remote): 添加 IPv6 DDNS 详细配置指南`

---
> Source: [cxbdasheng/AIO](https://github.com/cxbdasheng/AIO) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
