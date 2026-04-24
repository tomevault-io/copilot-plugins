## vibe-coding-tw

> `vibe-coding-cn` 專案旨在提供一個透過與 AI 結對程式設計實現“將想法變為現實”的終極工作流程。它強調“規劃驅動”和“模組化”的核心理念，旨在避免 AI 失控導致的專案混亂。該專案不僅僅是一個程式碼庫，更是一個全面的 AI 結對程式設計指南、龐大的提示詞庫和模組化的技能工具集，涵蓋了從專案構思、技術選型、實施規劃到具體開發、除錯和擴充套件的全過程。

# GEMINI.md - 專案上下文文件 (Project Context Document)

## 專案概述 (Project Overview)

`vibe-coding-cn` 專案旨在提供一個透過與 AI 結對程式設計實現“將想法變為現實”的終極工作流程。它強調“規劃驅動”和“模組化”的核心理念，旨在避免 AI 失控導致的專案混亂。該專案不僅僅是一個程式碼庫，更是一個全面的 AI 結對程式設計指南、龐大的提示詞庫和模組化的技能工具集，涵蓋了從專案構思、技術選型、實施規劃到具體開發、除錯和擴充套件的全過程。

**核心理念:** 規劃是核心，透過結構化、模組化的方式引導 AI，確保專案可控、可維護。

## 技術棧 (Technology Stack)

本專案主要的技術棧和相關工具包括：

*   **核心語言:** Python (用於 `prompts-library` 工具和備份指令碼)
*   **CLI 互動:** `rich`, `InquirerPy` (用於 `prompts-library` 提供友好的命令列介面)
*   **資料處理:** `pandas`, `openpyxl` (用於 `prompts-library` 處理 Excel 檔案)
*   **配置管理:** `PyYAML` (用於 `prompts-library` 的配置)
*   **文件規範:** `markdownlint-cli` (用於 `Makefile` 中的 `lint` 任務)
*   **版本控制:** Git
*   **自動化:** Makefile
*   **作業系統:** 相容 Linux

## 主要功能與工作流程 (Key Features & Workflow)

1.  **AI 提示詞庫 (`prompts/`):**
    *   一個極其龐大和精細分類的提示詞集合，是專案的核心資產。
    *   `coding_prompts/`: 專注於程式設計和程式碼生成的提示詞。
    *   `system_prompts/`: 用於設定 AI 行為和思維框架的系統級提示詞。
    *   `user_prompts/`: 使用者自定義或常用的提示詞。

2.  **提示詞庫管理工具 (`libs/external/prompts-library/`):**
    *   提供 Python 工具 (`main.py`)，用於在 Excel 工作簿 (`prompt_excel/`) 和 Markdown 文件 (`prompt_docs/`) 之間進行提示詞的相互轉換。
    *   支援互動式和非互動式操作。

3.  **技能庫 (`skills/`):**
    *   一個模組化的技能集合，為 AI 提供了特定工具和領域的知識。
    *   每個技能（如 `ccxt`, `postgresql`, `telegram-dev`）都包含獨立的 `SKILL.md` 描述, 參考資料和指令碼。

4.  **專案備份工具 (`backups/`):**
    *   `快速備份.py` 指令碼能根據 `.gitignore` 規則智慧地打包專案檔案為 `.tar.gz` 格式。

5.  **知識庫與文件 (`documents/`):**
    *   包含程式碼組織、開發經驗、系統提示詞構建原則、專案架構模板等各類文件。

6.  **外部工具與個人配置 (`libs/external/`):**
    *   存放非核心專案程式碼但有用的外部工具、個人配置或實驗性程式碼。例如：`my-nvim/` (nvim 配置), `XHS-image-to-PDF-conversion/` (圖片轉PDF工具)。

## 檔案結構 (File Structure)

```
.
├── .gitignore                   # Git 版本控制忽略檔案配置
├── AGENTS.md                    # 面向 AI Agent 的貢獻與行為準則。
├── CLAUDE.md                    # 面向 Claude 模型的上下文與指令。
├── CODE_OF_CONDUCT.md           # 專案行為準則。
├── CONTRIBUTING.md              # 貢獻指南。
├── GEMINI.md                    # 面向 Gemini 模型的上下文與指令 (本文件)。
├── LICENSE                      # 專案許可證。
├── Makefile                     # 專案自動化指令碼 (lint, backup 等)。
├── README.md                    # 專案主文件，包含專案概覽、使用指南等。
│
├── backups/                     # 專案備份指令碼。
│   ├── 一鍵備份.sh              # 一鍵備份指令碼。
│   └── 快速備份.py              # 快速備份 Python 指令碼。
│
├── documents/                   # 存放各類說明文件、經驗總結和配置。
│   └── ...                      # 更多文件內容。
│
├── libs/                        # 核心庫程式碼。
│   ├── common/                  # 通用功能和工具庫。
│   │   ├── __init__.py          # Python 包初始化檔案。
│   │   ├── models/              # 資料模型定義。
│   │   └── utils/               # 實用工具函式。
│   ├── database/                # 資料庫相關程式碼。
│   └── external/                # 外部工具、個人配置或實驗性程式碼。
│       ├── prompts-library/     # 提示詞庫管理工具 (Excel-Markdown 互轉)。
│       │   ├── main.py          # 提示詞庫管理工具主程式。
│       │   ├── requirements.txt # 工具依賴。
│       │   ├── prompt_excel/    # Excel 格式提示詞。
│       │   ├── prompt_docs/     # Markdown 格式提示詞文件。
│       │   └── ... (其他 prompts-library 內部檔案)
│       ├── my-nvim/             # 個人 Neovim 配置。
│       └── XHS-image-to-PDF-conversion/ # 小紅書圖片轉 PDF 工具。
│
├── prompts/                     # 核心資產：AI 提示詞庫。
│   ├── coding_prompts/          # 程式設計與程式碼生成相關提示詞。
│   ├── system_prompts/          # AI 系統級提示詞。
│   └── user_prompts/            # 使用者自定義提示詞。
│
└── skills/                      # 模組化技能庫。
    ├── ccxt/                    # CCXT 加密貨幣交易庫技能。
    ├── claude-code-guide/       # Claude Code 使用指南技能。
    ├── postgresql/              # PostgreSQL 資料庫技能。
    ├── telegram-dev/            # Telegram Bot 開發技能。
    └── ... (其他 10+ 個技能)
```

---
> Source: [godmakereth/vibe-coding-tw](https://github.com/godmakereth/vibe-coding-tw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
