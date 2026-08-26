## desktopkoalapet

> > **Windsurf/Cascade 使用者**：請先閱讀 `.github/skills/README.md` 了解所有可用的開發指南。

# KoalaPet Copilot 指導文件

> **Windsurf/Cascade 使用者**：請先閱讀 `.github/skills/README.md` 了解所有可用的開發指南。

## 重要：在執行任何修改前，請先閱讀相關 Skill 文件

當處理以下類型的任務時，**必須先讀取對應的 SKILL.md 文件**：

| 任務類型 | 對應 Skill 文件 |
|---------|----------------|
| 建立/修改 UI 視窗 | `.github/skills/koala-ui-window-creation/SKILL.md` |
| 建立新狀態 (State) | `.github/skills/koala-state-creation/SKILL.md` |
| 建立新管理器 (Manager) | `.github/skills/koala-manager-creation/SKILL.md` |
| 建立/修改物品 | `.github/skills/koala-item-creation/SKILL.md` |
| 翻譯/i18n | `.github/skills/koala-i18n-translation/SKILL.md` |
| 對話文案 | `.github/skills/koala-dialogue-authoring/SKILL.md` |
| 常數管理 | `.github/skills/koala-constants-management/SKILL.md` |
| 動畫整合 | `.github/skills/koala-animation-integration/SKILL.md` |
| 桌面物品渲染 | `.github/skills/koala-desktop-item-rendering/SKILL.md` |

## UI 視窗設計規範摘要

所有彈窗視窗必須遵循以下規範：

1. **無邊框**：使用 `Qt.WindowType.FramelessWindowHint`
2. **置頂**：使用 `Qt.WindowType.WindowStaysOnTopHint`
3. **透明背景**：`WA_TranslucentBackground`
4. **自定義關閉按鈕**：右上角的 ✕ 按鈕
5. **可拖曳**：實作 `mousePressEvent` / `mouseMoveEvent`
6. **進場動畫**：淡入效果

**禁止使用**：`WindowCloseButtonHint`、`WindowMinimizeButtonHint` 等系統按鈕

## 專案結構

- `core/` - 核心邏輯（KoalaPet 主體、狀態機）
- `states/` - 所有狀態類別
- `managers/` - 各種管理器
- `ui/` - UI 視窗元件
- `data/` - 常數、對話、物品資料、i18n
- `assets/` - 靜態資源

## 程式碼風格

- 使用繁體中文註解
- 遵循 PEP 8
- 類別/方法使用 docstring 說明

---
> Source: [knight0176/DesktopKoalaPet](https://github.com/knight0176/DesktopKoalaPet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
