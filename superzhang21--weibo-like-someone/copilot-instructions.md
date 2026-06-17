## weibo-like-someone

> Weibo Like Someone (微博模仿者)：深度克隆博主文风。支持抓取、分析、模仿生成全流程，支持 Agent 协作模式。


# Weibo Analyzer v6.5 — 微博博主克隆引擎

这是由 superzhang21 与赛博哈雷（Cyber Harley）共同开发的微博博主深度解构工具。它不仅能全自动完成“抓取-分析-生成”，还支持在无 API 环境下通过智能 Agent 协作完成创作。

## 🌟 核心特性

- **双模引擎**：支持自动化（LLM 驱动）和 Agent 协作（人工智能力量）两种模式。
- **高性能抓取**：基于微博桌面版 AJAX 接口，原生支持全文抓取，无需二次加载。
- **智能缓存**：内置 SQLite 数据库，特征分析结果 7 天内自动缓存。
- **数据闭环**：自动过滤转发，仅聚焦博主原创语料，确保文风克隆的纯净度。

## 🚀 快速开始

### 1. 配置 Cookie (燃料注入)
登录 [weibo.com](https://weibo.com)，在控制台执行 `document.cookie`，将结果保存到 `assets/cookies.json` (或通过脚本加密)。

### 2. 命令行操作
```bash
# 全流程：抓取 -> 分析 -> 模仿 (默认使用 OPENAI_API_KEY)
python3 cli.py --uid 1989660417 --action all --topic "评价人工智能"

# 纯抓取模式：仅获取语料并存入本地库
python3 cli.py --uid 1989660417 --action analyze --count 20

# Agent 协作模式：无 API Key 时，工具会自动输出语料供 Agent 参考
python3 cli.py --uid 1111681197 --action generate --topic "评价王思聪"
```

## 📂 目录结构
- `cli.py`: 统一入口
- `assets/weibo_scraper.py`: 桌面版 API 爬虫
- `assets/data_manager.py`: SQLite 存储管理
- `assets/feature_extractor.py`: 语言特征提取 (LLM)
- `assets/content_generator.py`: 文风模仿生成 (LLM)

---
*“技术应当服务于表达。把博主的灵魂代码化，就是对信息时代最好的致敬。”*

---
> Source: [superzhang21/weibo-like-someone](https://github.com/superzhang21/weibo-like-someone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
