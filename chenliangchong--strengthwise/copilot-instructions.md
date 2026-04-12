## strengthwise

> |------|------|------|

# UI 文檔更新規範

## 📂 文檔分工

| 文檔 | 用途 | 讀者 |
|------|------|------|
| `UI_DESIGN_SYSTEM.md` | 設計理念、決策原因 | 設計師、產品經理 |
| `UI_DEVELOPER_GUIDE.md` | 開發查表、快速參考 | 開發者 |

## 🔄 更新鏈

```
設計變更時的更新順序：
1. UI_DESIGN_SYSTEM.md  ← 修改設計理念
2. UI_DEVELOPER_GUIDE.md ← 更新對應數值
3. lib/themes/app_theme.dart ← 實作代碼
```

---

## 🎨 UI_DESIGN_SYSTEM.md 結構

```
UI_DESIGN_SYSTEM.md
├── 設計哲學（為何這樣設計）
├── 競品分析（借鏡什麼）
├── 色彩選擇理由（為何用這個藍色）
├── 字體選擇理由（為何用 Inter）
├── 網格系統理由（為何用 8 點）
├── 無障礙考量
└── 未來迭代方向
```

### 必須包含

- 每個設計決策的**理由**
- 競品對比（如何差異化）
- 設計原則（優先順序）

### 禁止包含

<critical>
- 具體 Hex 色碼（放 DEVELOPER_GUIDE）
- 具體尺寸數值（放 DEVELOPER_GUIDE）
- 代碼範例（放 DEVELOPER_GUIDE）
</critical>

---

## 📋 UI_DEVELOPER_GUIDE.md 結構

```
UI_DEVELOPER_GUIDE.md
├── 色彩快查表（Hex、變數名）
├── 字體快查表（大小、字重）
├── 間距快查表（8 點網格值）
├── 圓角/陰影快查表
├── 組件使用範例
└── 開發檢查清單
```

### 必須包含

- **可複製貼上**的數值
- 簡短代碼範例
- 淺色/深色對照表

### 格式規範

```markdown
## 色彩

| 用途 | 淺色模式 | 深色模式 | 變數 |
|------|---------|---------|------|
| Primary | `#2563EB` | `#60A5FA` | `colorScheme.primary` |
```

---

## ✅ 更新檢查清單

當修改 UI 設計時：

```
□ UI_DESIGN_SYSTEM.md - 更新設計理由（如有）
□ UI_DEVELOPER_GUIDE.md - 更新數值表
□ app_theme.dart - 更新代碼
□ 130-flutter-ui-widgets.mdc - 只有「規則變更」時才更新
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ChenLiangChong)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ChenLiangChong)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
