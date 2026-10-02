## cn-patent-creator

> 此文件为 Claude Code 在本项目中的工作提供指导。

# CLAUDE.md

此文件为 Claude Code 在本项目中的工作提供指导。

## 项目概述

`cn-patent-creator` 是一个中国发明专利申请文件全流程撰写 Skill。它融合了多个开源专利撰写技能的最佳实践，提供从技术交底书到 CNIPA 六文档申请文件包的一站式服务。

## 核心能力

| 能力 | 说明 |
|------|------|
| 技术交底书撰写 | 七章结构、Mermaid 附图渲染、.docx 输出 |
| 权利要求书撰写 | 独立权+从属权、方法+系统、引用合规 |
| 说明书撰写 | 六小节、[0001] 连续段号、充分公开 |
| 附图制作 | Mermaid 系统框图+流程图、PNG 渲染 |
| 摘要与请求书 | ≤300字摘要、IPC分类+诚信承诺 |
| 合规自检 | CNIPA 2026 10大类~50条核验 |

## 技术栈

- Markdown（撰写格式）+ python-docx（.docx 转换）
- Mermaid + mermaid-cli (mmdc)（附图渲染）
- mammoth（.docx → .md 转换）
- Bash 脚本（文件操作、格式转换）

## 目录约定

```
outputs/                           # 生成文件输出目录
├── {案件名}_{时间戳}.md          # 交底书 Markdown
├── {案件名}_{时间戳}.docx        # 交底书 Word
├── {案件名}_权利要求书_{时间戳}.md
├── {案件名}_权利要求书_{时间戳}.docx
├── {案件名}_说明书_{时间戳}.md
├── {案件名}_说明书_{时间戳}.docx
├── {案件名}_说明书摘要_{时间戳}.md
├── {案件名}_说明书摘要_{时间戳}.docx
├── {案件名}_专利请求书_{时间戳}.md
├── {案件名}_专利请求书_{时间戳}.docx
├── 附图1_系统组成结构示意图.png
└── 附图2_方法流程示意图.png
```

## 关键命令

```bash
# Mermaid 渲染为 PNG
mmdc -i input.mmd -o output.png -s 2 -b transparent

# Markdown 转 Word
python -c "
from docx import Document
# ... python-docx conversion
"

# Word 转 Markdown（扫描材料时）
python -c "
import mammoth
# ... mammoth conversion
"

# 字数统计（摘要合规检查）
wc -m abstract.md
```

## 注意事项

- 所有用户个人信息使用 `[占位]` 标记
- 时间戳格式：`YYYYMMDDHHmmss`
- 文件名含中文字符，Bash 操作时注意编码
- Windows 环境下 Chrome 路径需通过 `PUPPETEER_EXECUTABLE_PATH` 环境变量指定
- 输出文件默认由 `.gitignore` 忽略

---
> Source: [cjj197581/cn-patent-creator](https://github.com/cjj197581/cn-patent-creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
