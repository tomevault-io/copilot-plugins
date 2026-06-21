## asvt-n8blazorapp-tpl

> 這是一個基於 .NET 8 Blazor 的企業級 Web 應用程式範本專案，整合 MudBlazor UI 框架與 React Widgets，提供混合渲染模式（SSR + Interactive）的現代化 Web 開發基礎架構。專案採用多層式架構設計，包含自訂元件庫（Vista.Component）、業務邏輯層、資料存取層，適用於快速搭建具備認證機制、資料管理與可重用元件的 Web 應用程式。

# CLAUDE.md

## 專案快速摘要 (Overview)

這是一個基於 .NET 8 Blazor 的企業級 Web 應用程式範本專案，整合 MudBlazor UI 框架與 React Widgets，提供混合渲染模式（SSR + Interactive）的現代化 Web 開發基礎架構。專案採用多層式架構設計，包含自訂元件庫（Vista.Component）、業務邏輯層、資料存取層，適用於快速搭建具備認證機制、資料管理與可重用元件的 Web 應用程式。

---

## 技術棧 (Tech Stack)

### 核心框架 (Frameworks)
- **.NET 8.0** - Blazor Web App (支援 SSR + WebAssembly Interactive 模式)
- **MudBlazor 8.15** - Material Design UI 元件框架
- **React 18.x** - 前端元件擴展 (透過 ReactWidgets 專案)

### 程式語言 (Languages)
- **C# 12** (.NET 8)
- **JavaScript/TypeScript** (React 元件)
- **Razor** (Blazor 元件標記語言)

### 資料庫/狀態管理 (Data/State)
- **自訂資料庫抽象層** (Vista.DB / Vista.DbPanda)
- **ConnProxy** - 資料庫連線管理與保護機制
- **JWT 認證** (jose-jwt)

### 關鍵第三方工具/套件 (Key Libraries)
- **Serilog** - 結構化日誌記錄
- **FluentValidation** - 表單與模型驗證
- **BlazorComponentBus** - 元件間通訊匯流排
- **BlazorTemplater** - 範本引擎
- **Playwright** - 端到端測試 (ZPlaywrightTestProject)
- **Webpack** - React 元件打包

---

## 目錄結構導覽 (Project Structure)

```
Asvt_N8BlazorApp_TPL/
├── AsvtTPL/                          # 主 Web 應用程式專案
│   ├── Components/                   # Blazor 元件 (Pages, Layout, Account, Blog)
│   ├── Controllers/                  # Web API 控制器
│   ├── Models/                       # 資料模型與 ViewModels
│   ├── Services/                     # 應用程式服務層
│   ├── Template/                     # 範本檔案
│   ├── wwwroot/                      # 靜態資源 (CSS, JS, 圖片)
│   └── Program.cs                    # 應用程式進入點與 DI 容器設定
│
├── Vista.Component/                  # 可重用 Razor 元件庫 (v3.1.0)
│   └── *.razor                       # 自訂 Blazor UI 元件
│
├── Vista.Component.Abstractions/     # 元件抽象介面與契約
│
├── Vista.Biz/                        # 業務邏輯層
│   ├── Services/                     # 業務服務實作
│   ├── Models/                       # 業務實體模型
│   ├── DataPicking/                  # 資料擷取邏輯
│   └── DEMO/                         # 示範與測試用例
│
├── Vista.DB/                         # 資料存取層
│   ├── BasicData/                    # 基礎資料處理
│   ├── Schema/                       # 資料庫結構定義
│   ├── DBHelper.cs                   # 資料庫操作輔助工具
│   └── Utils.cs                      # 通用工具函式
│
├── Vista.DbPanda/                    # 資料庫連線管理模組
│   └── ConnProxy                     # 加密連線字串管理
│
├── ReactWidgets/                     # React 元件專案
│   ├── src/                          # React 元件原始碼
│   ├── webpack.config.js             # Webpack 打包設定
│   └── package.json                  # npm 套件設定
│
└── ZPlaywrightTestProject/           # 端到端測試專案
    └── *.cs                          # Playwright 測試腳本
```

---

## 主要功能與架構 (Key Features & Architecture)

### 系統運作邏輯
採用 **前後端混合渲染架構**，主應用程式使用 Blazor Server-Side Rendering (SSR) 提供初始頁面載入效能，並支援 WebAssembly Interactive 模式實現客戶端互動。React Widgets 作為獨立前端元件庫，透過 Webpack 打包後整合至 Blazor 應用程式中，提供特殊互動元件。

### 核心功能模組

1. **多層式架構設計**
   - 清晰的職責分離：Presentation (AsvtTPL) → Business Logic (Vista.Biz) → Data Access (Vista.DB)
   - 可重用元件庫 (Vista.Component) 支援跨專案共用

2. **資料庫抽象與安全管理**
   - 自訂 ConnProxy 實現資料庫連線字串加密與保護
   - DBHelper 提供統一資料存取介面

3. **認證與授權機制**
   - Cookie-based Authentication
   - JWT Token 支援
   - FluentValidation 整合表單驗證

4. **日誌與監控**
   - Serilog 結構化日誌記錄
   - 非同步日誌寫入 (Serilog.Sinks.Async)

5. **混合前端技術棧**
   - Blazor 元件負責主要業務介面
   - React Widgets 處理特殊互動需求
   - MudBlazor 提供一致的 Material Design 風格

---

## 開發指引

### 編碼標準
- **字元編碼**: 所有程式碼檔案一律使用 **UTF-8**
- **命名規範**: 遵循 C# 慣例 (PascalCase for types, camelCase for locals)

### 主要開發流程
1. **新增業務功能**: 在 Vista.Biz 建立服務 → AsvtTPL 註冊 DI → Components 建立 UI
2. **新增 UI 元件**: Vista.Component 專案新增 .razor 檔 → 發布 NuGet (可選) → AsvtTPL 引用
3. **React 元件整合**: ReactWidgets/src 開發 → webpack 打包 → AsvtTPL/wwwroot 引用

### 測試策略
- 使用 Playwright (ZPlaywrightTestProject) 進行端到端 UI 測試
- 業務邏輯單元測試應放置於 Vista.Biz 相關測試專案

---

**最後更新**: 2025-12-18
**專案版本**: AsvtTPL v0.0.1-alpha | Vista.Component v3.1.0

---
> Source: [relyky/Asvt_N8BlazorApp_TPL](https://github.com/relyky/Asvt_N8BlazorApp_TPL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
