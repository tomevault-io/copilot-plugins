## cogniscient

> 這是一個使用 Next.js、React 和 TypeScript 開發的網站專案。

# Gemini 專案設定：Cogniscient

## 1. 專案簡介

這是一個使用 Next.js、React 和 TypeScript 開發的網站專案。
(您可以在此處加入更多關於專案目標的描述)

## 2. 常用指令

請從 `package.json` 的 `scripts` 區塊確認以下指令是否正確。

- **安裝依賴**: `yarn`
- **啟動開發環境**: `yarn dev`
- **建置專案**: `yarn build`
- **執行測試**: `yarn test` (如果有的話)
- **程式碼檢查 (Linting)**: `yarn lint` (如果有的話)

## 3. 程式碼風格與慣例

- **語言**: 請使用 TypeScript。
- **風格**: 請遵循專案根目錄下的 `.prettierrc` 和 `eslint.config.mjs` 設定。
- **元件**:
  - 新的 React 元件請建立在 `src/components/` 目錄下。
  - 元件檔案請使用 `PascalCase.tsx` 命名 (例如: `NavBar.tsx`)。
  - 優先使用 const 關鍵字來宣告元件。
  - 優先使用 `const` 搭配箭頭函式來宣告元件。
- **CSS**:
  - 使用 Tailwind CSS 進行樣式設計。
  - 全域樣式請參考 `src/app/globals.css`。

## 4. 專案架構

- `src/app/`: Next.js 的 App Router 頁面和路由。
- `src/components/`: 可重用的 React 元件。
- `src/stores/`: 狀態管理 (State Management) 的 stores (例如 Zustand, Redux)。
- `public/`: 靜態資源，如圖片、SVG 等。
- `pages/api/`: API 路由 (如果仍在使用 Pages Router 的 API)。

## 5. 其他注意事項

- (任何您想讓 Gemini 知道的特別事項，例如：請勿直接使用 `<img>` 標籤，應使用 Next.js 的 `<Image>` 元件。)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sunpochin)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sunpochin)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
