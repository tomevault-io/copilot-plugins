## creation-store-analysis

> 這是一個使用 React + TypeScript + Vite 開發的倉儲租金分析系統，主要用於分析和展示 2024-2025 年的倉租變化趨勢、產品分類分析、新增/停用品項追蹤等功能。

# 倉租分析系統 - 開發指南

## 專案概述

這是一個使用 React + TypeScript + Vite 開發的倉儲租金分析系統，主要用於分析和展示 2024-2025 年的倉租變化趨勢、產品分類分析、新增/停用品項追蹤等功能。

## 技術架構

### 核心技術棧
- **前端框架**: React 19.1.0 + TypeScript 5.8.3
- **建置工具**: Vite 6.3.5
- **樣式框架**: Tailwind CSS 3.4.17
- **圖表庫**: Recharts 3.1.0
- **Excel 處理**: xlsx 0.18.5
- **工具函數**: Lodash 4.17.21

### 開發工具
- **包管理器**: npm
- **代碼品質**: TypeScript 嚴格模式
- **樣式處理**: PostCSS + Autoprefixer
- **部署**: GitHub Pages + GitHub Actions

## 專案結構

```
creation-store-analysis/
├── public/
│   └── data-07-19.xlsx          # Excel 數據文件
├── src/
│   ├── components/
│   │   └── WarehouseAnalysisSystem.tsx  # 主要分析組件
│   ├── App.tsx                  # 應用程式根組件
│   ├── main.tsx                 # 應用程式入口點
│   └── index.css               # 全域樣式 + Tailwind 導入
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions 自動部署
├── dist/                       # 建置輸出目錄
├── package.json               # 專案依賴和腳本
├── vite.config.ts            # Vite 配置
├── tailwind.config.js        # Tailwind CSS 配置
├── postcss.config.js         # PostCSS 配置
├── tsconfig.json             # TypeScript 配置
└── README.md                 # 專案說明文件
```

## 核心功能模組

### 1. 數據載入與處理 (`loadAndProcessData`)
- **檔案來源**: `/public/data-07-19.xlsx`
- **工作表**: `合併`、`2025年6月庫存量`、`2024年6月庫存量`
- **環境適應**: 自動檢測開發/生產環境調整檔案路徑
- **錯誤處理**: 完整的 try-catch 包裝和用戶友好的錯誤訊息

### 2. 數據分析引擎
#### `calculateDetailedSummary` - 產品別摘要分析
- 按中分類分組統計 2024/2025 年倉租變化
- 計算變化金額和變化率
- 小分類按變化金額排序（正值優先）
- 過濾原料分類，取前 10 項

#### `calculateWarehouseAnalysis` - 外倉分析
- 計算各外倉（大昌華嘉、豐安、大榮、川田、成功、宗運）金額分布
- 分析庫存數量與倉租金額的關聯性
- 計算佔比數據（金額佔比、庫存佔比）

#### `calculateNewItems` - 新增品項分析
- 識別 2025 年新出現的料號
- 結合庫存數量數據
- 按倉租金額排序

#### `calculateDiscontinuedItems` - 停用品項分析
- 識別 2024 年有但 2025 年消失的料號
- 保留 2024 年的庫存和倉租數據

### 3. 視覺化組件
#### 圓餅圖分析 (`activeTab: 'pie'`)
- 使用 Recharts PieChart 展示 2025 年倉租佔比
- 漸層色彩設計 + 陰影效果
- 互動式點選查看小分類明細
- 動態計算百分比和金額

#### 產品別統計表 (`activeTab: 'table'`)
- 可展開的表格設計
- **特殊樣式**: 變化金額 > 50,000 的行顯示淡黃色背景
- 趨勢圖標（上升/下降/持平）
- 圓形展開按鈕 + 旋轉動畫
- 完整的響應式設計

#### 外倉分析頁面
- 卡片式摘要展示各外倉總計
- 可展開表格顯示小分類詳細資訊
- 進度條展示佔比數據

## 重要技術細節

### 1. 路徑處理邏輯
```typescript
// 環境檢測 - 確保在不同環境下正確載入檔案
const isProduction = window.location.hostname === 'vitochiang.github.io';
const basePath = isProduction ? '/creation-store-analysis/' : '/';
const response = await fetch(`${basePath}data-07-19.xlsx`);
```

### 2. Excel 讀取配置
```typescript
// XLSX 讀取選項 - 注意是 cellFormula 不是 cellFormulas
const workbook = XLSX.read(arrayBuffer, { 
  cellStyles: true, 
  cellFormula: true,  // ⚠️ 不是 cellFormulas
  cellDates: true, 
  cellNF: true, 
  sheetStubs: true 
});
```

### 3. 狀態管理架構
- 主數據狀態: `data` (處理後的分析結果)
- UI 狀態: `loading`, `error`, `activeTab`
- 展開狀態: `expandedCategories`, `expandedWarehouse2024/2025`
- 選中狀態: `selectedPieCategory`

### 4. 展開/收合邏輯
```typescript
// 統一的展開控制函數
const toggleExpand = (category: string, type: string) => {
  const setters: Record<string, any> = { 
    categories: setExpandedCategories, 
    warehouse2024: setExpandedWarehouse2024, 
    warehouse2025: setExpandedWarehouse2025 
  };
  setters[type]((prev: Record<string, boolean>) => ({ 
    ...prev, 
    [category]: !prev[category] 
  }));
};
```

## 樣式設計指南

### 1. 顏色系統
- **主色調**: 藍色系 (blue-500, blue-600)
- **成功色**: 綠色 (green-500, green-600) - 減少變化
- **警告色**: 紅色 (red-500, red-600) - 增加變化
- **中性色**: 灰色系 (gray-50 到 gray-800)
- **特殊標示**: 淡黃色 (yellow-50) - 重要變化標示

### 2. 互動元素
- **懸停效果**: `hover:bg-gray-50` 等
- **過渡動畫**: `transition-all duration-200`
- **展開動畫**: `rotate-180` 轉換
- **響應式**: `grid-cols-2 md:grid-cols-3 lg:grid-cols-6`

### 3. 表格設計
- **斑馬紋**: `divide-y divide-gray-200`
- **懸停高亮**: 整行變色
- **數字對齊**: 右對齊 (`text-right`)
- **置中元素**: 使用 `flex justify-center`

## 部署配置

### 1. GitHub Pages 自動部署
- **觸發**: 推送到 `main` 分支
- **權限**: `contents: read`, `pages: write`, `id-token: write`
- **環境**: `github-pages`
- **構建**: Node.js 18 + npm

### 2. Vite 配置
```typescript
// 環境適應的 base 路徑
export default defineConfig({
  plugins: [react()],
  base: process.env.NODE_ENV === 'production' 
    ? '/creation-store-analysis/' 
    : '/',
});
```

### 3. package.json 關鍵配置
- `"type": "module"` - 支援 ES6 模組
- `"homepage"` - GitHub Pages 路徑
- 部署腳本: `predeploy` + `deploy`

## 常見問題與解決方案

### 1. 檔案載入問題
**問題**: 404 錯誤找不到 Excel 檔案
**解決**: 確認檔案在 `public/` 目錄，檢查路徑邏輯

### 2. TypeScript 錯誤
**問題**: `import.meta.env` 類型錯誤
**解決**: 使用 `(import.meta as any).env` 或檢查 `vite/client` 類型

### 3. 樣式不生效
**問題**: Tailwind CSS 不工作
**解決**: 確認 `postcss.config.js` 使用正確語法，檢查 `type: module`

### 4. 部署權限問題
**問題**: GitHub Actions 403 錯誤
**解決**: 使用官方 `actions/deploy-pages@v4`，設定正確權限

### 5. 數據格式問題
**問題**: Excel 讀取後數據不正確
**解決**: 檢查工作表名稱（中文編碼）、欄位名稱、數據類型

## 開發最佳實務

### 1. 數據處理
- 使用 Lodash 進行複雜的數據操作
- 總是檢查數據存在性 (`item['料號'] && typeof item['期末存量'] === 'number'`)
- 使用 `Math.round()` 處理金額顯示

### 2. 組件設計
- 保持組件功能單一性
- 使用 TypeScript 嚴格類型檢查
- 適當的錯誤邊界和載入狀態

### 3. 效能優化
- 使用 `React.Fragment` 減少 DOM 節點
- 適當的 key 值設定
- 避免在 render 中進行複雜計算

### 4. 用戶體驗
- 提供載入狀態和錯誤提示
- 響應式設計支援多裝置
- 直觀的互動反饋（懸停、點擊效果）

## 維護注意事項

1. **數據更新**: 更換 Excel 檔案時注意工作表名稱和欄位結構
2. **依賴升級**: 定期檢查和更新依賴包，特別注意 breaking changes
3. **瀏覽器兼容**: 測試主流瀏覽器支援性
4. **效能監控**: 關注 bundle 大小和載入速度
5. **安全性**: 不要在代碼中硬編碼敏感信息

## 擴展建議

1. **數據來源**: 考慮支援多種數據格式 (CSV, JSON API)
2. **導出功能**: 加入 PDF/Excel 導出功能
3. **篩選排序**: 增加更多數據篩選和排序選項
4. **比較分析**: 支援多時期數據比較
5. **用戶偏好**: 加入個人化設定保存

---

*最後更新: 2025-07-19*
*開發者: Claude & VitoChiang*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VitoChiang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
