## web-content-summarizer

> *本项目的大部分指令、参数和处理流程已沉淀至 `SKILL.md` 中。此处仅保留项目级别的上下文补充。*

# Web Content Summarizer - Agent 记忆

*本项目的大部分指令、参数和处理流程已沉淀至 `SKILL.md` 中。此处仅保留项目级别的上下文补充。*

## 硬件自适应说明

- **GPU**: 推荐配置为 `large-v3` + `float16`
- **CPU**: 推荐配置为 `base` + `int8`

## 目录结构

```text
web-content-summarizer/
├── SKILL.md        ← Skill 定义（包含核心处理流程、参数和使用方式）
├── PROMPT.md       ← 笔记总结模板（包含输出的 Markdown 格式要求）
├── AGENTS.md       ← 本文件（项目上下文补遗）
├── src/            ← 源代码
│   ├── main.py
│   ├── transcribe_url.py
│   ├── downloader.py
│   ├── audio.py
│   ├── subtitles.py
│   └── transcriber.py
├── temp/           ← 临时文件
└── venv/           ← 虚拟环境
```

---
> Source: [pbdm/web-content-summarizer](https://github.com/pbdm/web-content-summarizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
