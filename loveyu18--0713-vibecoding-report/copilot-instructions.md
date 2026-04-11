## 0713-vibecoding-report

> 這是一個關於「Vibe Coding」並評估前端開發中 AI 工具效率的研究專案。

# Gemini 專案背景：Vibe Coding 與 AI 工具研究

這是一個關於「Vibe Coding」並評估前端開發中 AI 工具效率的研究專案。

## 專案概覽

- **主要目標：** 研究、記錄並報告關於使用 AI 提升前端開發工作流程的發現。
- **核心概念：** 「Vibe Coding」似乎是本專案正在探索的核心方法論或實踐。
- **語言：** 專案包含繁體中文和英文的檔案與內容。在後續的互動中，請維持這種雙語特性。

## 主要目錄

- `documents/`: 包含核心研究資料，如指南、大綱和週報。
- `presentations/`: 存放用於分享研究成果的 HTML 簡報。
- `projects/`: 包含特定的案例研究或子專案，例如 `amazon-kiro-ide` 的探索。
- `.claude/`: 包含 Claude AI 助理的設定和提示，顯示此研究採用了多種 AI 工具。
- `claudeCode/`: 包含與 Claude AI 相關的簡報和程式碼。
- `cleanup-test/`: 包含一個關於海灘清潔活動的設計原型或測試專案。

## 設計規範 (Design Specification)

- **風格主軸:** 遵循 Apple 的 "iOS Liquid Glass" (Glassmorphism) 風格。
- **核心原則:**
    - **穿透感 (Translucency):** 使用半透明背景和背景模糊效果 (`backdrop-filter`) 來創造層次感。
    - **圓角:** 所有卡片、容器等元素都應使用明顯的圓角 (例如 `border-radius: 16px;`)。
    - **細緻邊框:** 使用 1px 的淺色半透明邊框來定義元素輪廓，增加玻璃質感。
    - **簡潔排版:** 使用清晰、易讀的無襯線字體 (如 `system-ui`, `sans-serif`)。
- **CSS 實作參考:**
  ```css
  .glass-container {
    background: rgba(255, 255, 255, 0.2); /* 半透明背景 */
    backdrop-filter: blur(10px) saturate(180%);
    -webkit-backdrop-filter: blur(10px) saturate(180%); /* 支援 Safari */
    border-radius: 16px;
    border: 1px solid rgba(255, 255, 255, 0.3);
    box-shadow: 0 8px 32px 0 rgba(31, 38, 135, 0.1); /* 可選的柔和陰影 */
  }
  ```

## 給 Gemini 的指示

- 當建立或修改檔案時，請遵循現有的雙語命名慣例和內容風格。
- **在設計 HTML 簡報或任何視覺介面時，請務必遵守上述的 "iOS Liquid Glass" 設計規範。**
- 主要的內容格式是 Markdown (`.md`) 文件和 HTML (`.html`) 簡報。
- 請理解此專案以研究為導向，並專注於清晰度、文件記錄和分析。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Loveyu18)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Loveyu18)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
