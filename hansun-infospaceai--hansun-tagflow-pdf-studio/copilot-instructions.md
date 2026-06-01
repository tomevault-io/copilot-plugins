## hansun-tagflow-pdf-studio

> HanSun TagFlow PDF Studio 是一個繁體中文 Windows PDF 桌面工具。主要技術為 Python、Tkinter、PyMuPDF、Pillow、python-docx、openpyxl、python-pptx、PyInstaller 與 Inno Setup。

# AGENTS.md

## Project Overview

HanSun TagFlow PDF Studio 是一個繁體中文 Windows PDF 桌面工具。主要技術為 Python、Tkinter、PyMuPDF、Pillow、python-docx、openpyxl、python-pptx、PyInstaller 與 Inno Setup。

本專案採 personal-use source-available license。任何修改不得移除非商業使用限制、品牌聲明或授權條款。

## Repository Layout

- `src/pdf_app/`：主要 Python 程式
- `assets/`：圖示與安裝介面圖片
- `installer/`：Inno Setup 安裝腳本
- `docs/`：GitHub、建置、功能與授權文件
- `.github/`：Issue / PR templates
- `dist/`：本機建置輸出位置，不應提交 exe
- `build/`：本機建置暫存，不應提交

## Setup

```bash
python -m venv .venv
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python run.py --self-test
```

## Validation Commands

```bash
python run.py --self-test
python -m compileall src run.py
```

Windows build:

```bat
BUILD_OFFICIAL_SETUP_EXE_ONE_CLICK.bat
```

## Coding Rules

- 保持繁體中文 UI。
- 不要加入簡體中文 UI 字串。
- 不要加入 PDF 破解、繞過權限、移除保護功能。
- 不要上傳私人 PDF、客戶資料、API key、token、授權碼或 build log。
- 不要提交 `dist/`, `build/`, `.venv/`, `*.exe`, `*.spec`，除非任務明確要求。
- 介面修改要注意右側面板文字不可被吃字，長檔名需換行且對齊。
- 維持 NoTrial 個人使用模式，不恢復試用倒數與授權碼阻擋。

## File Safety

- PDF 編輯功能不可覆蓋原檔，除非使用者明確選擇儲存。
- 批次處理預設輸出到新檔或新資料夾。
- 任何清理舊關聯/registry 的腳本都要明確提示需要系統管理員權限。

## Completion Report

修改後請回報：

1. Summary
2. Changed files
3. Commands run
4. Test results
5. Assumptions
6. Remaining risks
7. Manual QA steps

---
> Source: [Hansun-infospaceai/Hansun-Tagflow-PDF-Studio](https://github.com/Hansun-infospaceai/Hansun-Tagflow-PDF-Studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
