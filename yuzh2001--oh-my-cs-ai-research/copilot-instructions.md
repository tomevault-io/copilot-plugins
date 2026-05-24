## oh-my-cs-ai-research

> 2. 这是科研讨论工作区，默认目标是检索、精读、比较、整理、写分析，不把这里当成常规代码仓库使用。除非用户明确提出，否则不创建工程脚手架，不补测试框架，不围绕构建与发布流程展开工作。

1. 永远使用中文对话。
2. 这是科研讨论工作区，默认目标是检索、精读、比较、整理、写分析，不把这里当成常规代码仓库使用。除非用户明确提出，否则不创建工程脚手架，不补测试框架，不围绕构建与发布流程展开工作。

## 目录约定

- 所有持续性产物都放进 `jobs/yymmdd/hhmm-主题/`。
- 根目录默认只保留 `jobs/`、`AGENTS.md`、隐藏配置目录与少量确有必要的全局文件。
- 新 work 建议按需使用这些子目录：
  - `references/paper/`
  - `references/report/`
  - `notes/`
  - `outputs/`
  - `artifacts/`
- 临时文件优先放在对应 work 目录内部；只有跨 work 复用的临时状态才放根目录隐藏目录。

## 文献与网页处理

- 涉及 PDF、扫描件、网页抽取时，优先使用 [$convert-pdf-to-markdown-mineru](/Users/yuzh/.agents/skills/research/convert-pdf-to-markdown-mineru/SKILL.md)。
- 如果环境里的 MinerU 已经配置过 auth token。默认优先使用 `mineru-open-api extract`，不要把 `flash-extract` 作为默认路径；只有在用户明确要求更快的轻量抽取，或者 `extract` 当前不可用时，才考虑 `flash-extract`。
- MinerU 输出优先落到当前 work 目录内的 `artifacts/`、`references/` 或 `outputs/`，不要散落到仓库根目录。
- 遇到 IEEE 论文时，无论当前能否抓到全文，都要把 IEEE 页面链接或 DOI 链接一并发给用户，留作手动下载入口。
- 网页获取顺序保持为：内置 `r.jina.ai`、`curl`。

## 交互工具

- 需要可视化界面操作时，允许使用 Codex 的 computer-use 能力。

---
> Source: [yuzh2001/oh-my-cs-ai-research](https://github.com/yuzh2001/oh-my-cs-ai-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-24 -->
