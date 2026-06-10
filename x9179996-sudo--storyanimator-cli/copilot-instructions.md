## storyanimator-cli

> PDF → MinerU 解析 → LLM 旁白腳本 → edge-tts + moviepy 動態影片。

# StoryAnimator-CLI

PDF → MinerU 解析 → LLM 旁白腳本 → edge-tts + moviepy 動態影片。

## 核心限制（不可破壞）

- MinerU 3.1.14 呼叫用 `mineru.exe` 直接呼叫（`python -m mineru` 在此版本不支援）
- MinerU 輸出路徑結構：`{out_dir}/{pdf_stem}/auto/{pdf_stem}.md`
- moviepy 鎖 1.0.3，不升級
- 語音固定 zh-TW-HsiaoChenNeural
- 所有輸出目錄沿用 YYYYMMDD_任務名稱 命名

## 自動執行規則

所有 bash 和 PowerShell 指令都自動執行，不需要每次確認。
遇到真正的技術阻礙（缺少 API Key、檔案不存在）才停下來報告。

## 入口

`python story_animator.py --pdf <path> --title <title>`

## 測試 PDF

`C:\Users\JinYi\mineru_test\教材.pdf`

## Agent skills

### Issue tracker

Issues live as local markdown files under `.scratch/<feature-slug>/`. See `docs/agents/issue-tracker.md`.

### Triage labels

Five canonical roles; all label strings match the mattpocock/skills defaults. See `docs/agents/triage-labels.md`.

### Domain docs

Single-context layout: one `CONTEXT.md` + `docs/adr/` at the repo root. See `docs/agents/domain.md`.


## Agent View 任務清單

- 圖片管線驗證：claude --bg "驗證圖片管線，從 content_list.json 的 page_idx 對應到實際圖片"
- script_gen.py 改造：claude --bg "將 script_gen.py 改為 block 漸進式讀取，加入 retry 邏輯"
- 三種輸出格式：claude --bg "加入課堂 16:9、家長直式、社群短片三種輸出格式"


## 執行紀錄

---
> Source: [x9179996-sudo/StoryAnimator-CLI](https://github.com/x9179996-sudo/StoryAnimator-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
