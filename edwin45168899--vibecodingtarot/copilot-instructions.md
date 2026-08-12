## vibecodingtarot

> 塔羅牌占卜網站 — Python Flask web application for tarot card divination.

# VibeCodingTarot - Gemini Code Assist 指南

## 專案概述

塔羅牌占卜網站 — Python Flask web application for tarot card divination.

- **Core Purpose**: 線上塔羅牌占卜服務，提供單張與三張牌陣（過去-現在-未來）
- **Tech Stack**: Python Flask, HTML5/CSS3/JavaScript ES6+, JSON 資料存儲
- **Language**: 使用者介面為繁體中文

## 重要規則

### 1. 任務管理
- **確認任務 = 查 GitHub Issues**：使用者說「確認任務」時，執行 `gh issue list` 查詢，不使用 task 工具追蹤

### 2. Git 工作流程
- **PR Workflow**: 所有程式碼變更必須透過 Pull Requests（使用者偏好：「我想用PR的方式執行」）
- **Commit 格式**: `<type>(<scope>): <subject>`，主旨與內容均為**繁體中文**

### 3. 執行優先序
Security → Architecture → Code Quality → Performance → Features

## 架構決策

1. **JSON 資料存儲**: 無需資料庫，易於編輯/擴展塔羅牌資料
2. **Flask 框架**: 輕量級 Python web framework
3. **Single-Page Templates**: 每種占卜類型獨立 HTML 模板
4. **RESTful API**: 一致的 JSON 回應格式 `{"success": bool, "data": {...}, "message": "string"}`
5. **Purple Gradient Theme**: `linear-gradient(#667eea → #764ba2)` 神秘美學

## API Endpoints

- `POST /api/draw-single` — 抽單張牌
- `POST /api/draw-three` — 抽三張牌（過去-現在-未來）
- `GET /api/card/<card_id>` — 取得單張牌詳情
- `POST /api/save-reading` — 儲存占卜結果

## 已知問題

- Template 重複：index.html, single_card.html, three_cards.html 重複 header/footer 及 JS 邏輯
- 目前僅實作 22 Major Arcana；56 Minor Arcana 及 Celtic Cross spread 尚未實作

## GitHub Issues

- #1 Security (bug)
- #2 Architecture (enhancement)
- #3 Performance (enhancement)
- #4 Code Quality (enhancement)
- #5 Feature Gaps (enhancement)

## 協作規範

- **語言**: 回覆與 Commit 統一使用**繁體中文**
- **互動**: 主動澄清、引導思考
- **DoD**: 通過 Lint、更新 CHANGELOG、遵循 Commit 格式

---
> Source: [edwin45168899/VibeCodingTarot](https://github.com/edwin45168899/VibeCodingTarot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
