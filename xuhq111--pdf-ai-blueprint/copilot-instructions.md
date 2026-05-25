## pdf-ai-blueprint

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

基于AI Agent的PDF工程图文字识别系统MVP。输入PDF工程图纸（线束图、电路原理图等），输出图纸中元器件名称、端子号、表格内容的结构化文本。

## 技术栈

- Python 3.10+
- PDF解析：PyMuPDF (fitz)
- 图像处理：Pillow + numpy
- AI模型：`qwen-omni-vl-plus`，通过 DashScope `MultiModalConversation.call()` 调用
- 表格OCR备选：PaddleOCR
- 环境隔离：virtualenv 或 conda

## 架构与处理流程

管道式架构，5个模块按顺序执行：

```
PDF文件 → pdf_to_image → locate_regions → crop_images → recognize_text → merge_results → JSON输出
```

### 模块职责

| 模块 | 职责 | 关键细节 |
|------|------|----------|
| `pdf_to_image` | PDF单页→高分辨率PNG（≥300 DPI） | PyMuPDF |
| `locate_regions` | 降采样后调用大模型定位文字区域，输出归一化bbox | 降采样至≤4MP，JSON mode |
| `crop_images` | 归一化坐标→原图像素坐标，切割小图+20px padding | 反向映射公式见下方 |
| `recognize_text` | 逐小图识别文字，表格区域走PaddleOCR，否则走大模型 | 表格返回二维数组 |
| `merge_results` | 汇总为统一结构JSON输出 | 结构见下方 |



## MVP范围

- 仅支持单页PDF处理（接口保留多页扩展能力）
- 无需持久化存储、前端界面、并发处理
- 命令行调用，结果输出到控制台或JSON文件

## 关键约定

- 所有中文使用UTF-8编码
- 内存敏感：处理10000×14000像素图时峰值不超过4GB
- 代码模块化设计，便于未来替换定位/识别模型
- 表格区域优先走PaddleOCR，未配置时退化为大模型逐行识别并记录warning

---
> Source: [XuHq111/pdf-ai-blueprint](https://github.com/XuHq111/pdf-ai-blueprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
