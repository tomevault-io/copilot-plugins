## fable5-mythos5-zh

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

本项目是 Anthropic 于 2026-06-09 发布的 319 页 ["Claude Fable 5 and Mythos 5 System Card"](https://anthropic.com) PDF 的中文翻译项目。这不是一个代码仓库——它包含从 PDF 提取的文本、中文翻译成果以及最终的翻译版 PDF。

## 文件说明

| 文件 | 用途 |
|------|------|
| `Claude Fable 5 and Mythos 5 System Card.pdf` | 原始英文 PDF（27MB） |
| `Claude Fable 5 and Mythos 5 System Card_ZH.pdf` | 最终中文翻译 PDF（26.8MB） |
| `en-full-text.md` | 从 PDF 提取的完整英文文本（~610KB），按页码分段 |
| `texts.json` | 待翻译的英文文本片段数组，9,152 条 |
| `translations.json` | `{"英文文本": "中文翻译"}` 的键值对，9,152 条 |
| `zh-full-text.md` | 完整中文翻译，与 `en-full-text.md` 页面对齐 |
| `zh-executive-summary.md` | 执行摘要中文翻译 |
| `zh-chapters-1-3.md` / `zh-chapters-4-5.md` / `zh-chapters-6-7.md` / `zh-chapters-8-9.md` | 按章节分组的中文翻译 |
| `zh-summary.md` | 中文结构化摘要（含表格对比） |

## 数据模型

- `texts.json` — 字符串数组，每个元素是从 PDF 中提取的一个英文文本片段（段落、表格单元格、标题等）
- `translations.json` — 对象，key 为 `texts.json` 中的英文原文，value 为对应的中文翻译。两个文件条目一一对应（各 9,152 条）

## 常见操作

- **校对翻译**: 对比 `en-full-text.md` 与 `zh-full-text.md`，或检查 `translations.json` 中特定条目
- **更新中文 PDF**: 如果重新生成翻译版 PDF，使用 `translate-pdf` skill 处理原始 PDF，并以 `translations.json` 作为术语参考
- **提取未翻译片段**: 比较 `texts.json` 和 `translations.json` 的 key 集合，找出缺失项

---
> Source: [zhenghengzhi/fable5-mythos5-zh](https://github.com/zhenghengzhi/fable5-mythos5-zh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
