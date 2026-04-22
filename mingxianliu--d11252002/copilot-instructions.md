## d11252002

> - **專案名稱**: MXL (mingxianliu.github.io)

# MXL 專案開發指南

## 📋 專案概覽
- **專案名稱**: MXL (mingxianliu.github.io)
- **專案類型**: 個人網站/互動式命令指南
- **技術棧**: HTML, CSS (Tailwind), JavaScript, Chart.js
- **主要功能**: 提供 Claude Code 斜線命令的互動式指南

## 🎯 開發目標
建立一個完整的 Claude Code 斜線命令系統，包括：
- 專案級命令配置
- 個人級命令管理
- 安全權限控制
- 效能優化工具
- 程式碼品質檢查

## 📁 專案結構
```
mxl/
├── .claude/                    # Claude Code 配置
│   ├── settings.json          # 專案設定
│   ├── commands/              # 專案級命令
│   │   ├── optimize.md       # 效能最佳化
│   │   ├── review.md         # 程式碼審查
│   │   ├── security-audit.md # 安全審計
│   │   └── commit.md         # Git 提交
│   └── hooks/                # 生命週期 hooks
│       └── pre-tool-use.js   # 工具使用前檢查
├── index.html                 # 主要頁面
├── README.md                  # 專案說明
└── CLAUDE.md                  # 專案記憶檔案
```

## 🔧 開發規範

### 程式碼風格
- 使用繁體中文註解
- 遵循 ESLint 規範
- 保持程式碼簡潔和可讀性
- 添加詳細的功能說明

### Git 提交規範
遵循 Conventional Commits 格式：
```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

### 安全規範
- 禁止執行危險命令（rm -rf, sudo 等）
- 保護敏感檔案和目錄
- 驗證所有輸入資料
- 記錄所有工具使用情況

## 🚀 可用命令

### 專案級命令
- `/optimize` - 程式碼效能最佳化
- `/review` - 全面程式碼審查
- `/security-audit` - 安全漏洞檢查
- `/commit` - 自動化 Git 提交

### 個人級命令
- `/performance-check` - 深入效能分析

## 📊 效能指標
- 頁面載入時間 < 2 秒
- 程式碼執行效率優化
- 記憶體使用最小化
- 使用者體驗流暢

## 🔒 安全要求
- 輸入驗證和清理
- XSS 防護
- CSRF 保護
- 安全的資料處理

## 📈 品質標準
- 程式碼覆蓋率 > 80%
- 無嚴重安全漏洞
- 符合無障礙標準
- 響應式設計支援

## 🛠️ 開發工具
- Claude Code 作為主要開發助手
- 使用斜線命令提高開發效率
- 自動化測試和部署流程
- 持續整合和部署 (CI/CD)

## 📝 更新日誌
- 2025-01-28: 建立專案基礎結構
- 2025-01-28: 配置 Claude Code 命令系統
- 2025-01-28: 建立安全權限控制機制

---

*此檔案由 Claude Code 自動維護，請勿手動修改* 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mingxianliu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
