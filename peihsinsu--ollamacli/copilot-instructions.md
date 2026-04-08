## ollamacli

> 必須主動呼叫 progress-recorder agent 來記錄重要決策、任務變更、完成事項等關鍵資訊到 progress.md

# Ollamacli 專案配置

## 專案記憶規則
必須主動呼叫 progress-recorder agent 來記錄重要決策、任務變更、完成事項等關鍵資訊到 progress.md

偵測到以下情況時 **立即自動觸發** progress-recorder：
- 出現「決定使用／最終選擇／將採用」等決策語言
- 出現「必須／不能／要求」等約束語言
- 出現「完成了／實現了／修復了」等完成標識
- 出現「需要／應該／計劃」等新任務
- 重要架構或技術選型變更
- 關鍵問題解決方案確定

當 progress.md 的 Notes/Done 條目過多（>30 條）影響閱讀時，應歸檔到 progress.archive.md

## 專案資訊

### 基本資訊
- **專案名稱**: ollamacli
- **專案類型**: CLI Application
- **技術棧**: Golang
- **AI 引擎**: 無
- **PRD 文件**: [docs/design.md](./docs/design.md)

### 開發約束
- 使用 Go 1.21+ 版本
- 遵循 Go 代碼規範和最佳實踐
- 使用 Cobra 框架進行 CLI 開發
- 採用依賴注入模式
- 單元測試覆蓋率 > 80%

### 開發完成規範
- 開發完成請執行測試流程，使用所建立的 Makefile 進行觸發啟動與相關測試
- 有資料庫之專案，請在 Makefile 中加入資料庫的 init 動作以及測試資料的建立
- 凡 RESTful API 服務專案，請提供 /api/docs 頁面，提供 API 文件與測試，並使用 Swagger 工具產生文件，並提供 REST Client 格式測試頁於 test 資料夾下

### 架構原則
- 採用分層架構（CLI-Service-Client）
- CLI 指令設計遵循 Unix 工具慣例
- 實施完整的錯誤處理機制
- 結構化日誌記錄
- 配置管理分離（環境變數、設定檔）

## 指令集

### 核心指令
- **record**: 使用 progress-recorder 執行增量合併任務
- **archive**: 使用 progress-recorder 執行快照歸檔任務
- **recap**: 閱讀 progress.md，回顧專案當前狀態
- **doc**: 檢查文件，確認功能已完整描述
- **todo**: 檢查TODO.md檔案未完成事項(有[]符號的項目)並執行，完成後將項目搬到"# Complete"下並修改標示為[*]，並在最後押上完成時間

### 開發指令
- **build**: `make build` 或 `go build ./cmd/...`
- **run**: `make run` 或 `go run cmd/main.go`
- **test**: `make test` 或 `go test ./...`
- **lint**: `make lint` 或 `golangci-lint run`
- **setup**: `make setup` (完整環境設定)

## 目錄結構

### 核心目錄
- **docs/**: 技術文件與設計文件
  - **decisions/**: 架構決策記錄 (ADR)
  - **api/**: API 文件 (如適用)
  - **development/**: 開發指南
- **scripts/**: 自動化腳本
- **test/**: 測試資料與配置
- **samples/**: 範例資料
- **.claude/**: Claude 相關配置

### 程式碼結構
- **cmd/**: CLI 子指令定義與入口點
- **internal/**: 內部業務邏輯
  - **config/**: 設定管理 (YAML、環境變數)
  - **client/**: HTTP 客戶端與 Ollama API 封裝
  - **chat/**: 互動式對話處理
  - **output/**: 輸出格式化 (文字、JSON、串流)
  - **log/**: 統一 logging 介面
- **pkg/**: 可重用的套件
- **examples/**: 使用範例

## CLI 特定架構

### 主要子指令
- `list`: 列出可用模型
- `pull`: 拉取模型
- `show`: 顯示模型資訊
- `chat`: 互動式對話 (支援 --interactive 持續 session)
- `run`: 一次性推論
- `serve`: 本地服務模式

### Interactive Mode 特性
- **持續對話**: `ollamacli chat <model> --interactive`
- **上下文保持**: 自動維護對話歷史
- **優雅退出**: Ctrl+C 清理資源後離開
- **Readline 支援**: 指令歷史、自動完成
- **Session 指令**: `/help`, `/clear`, `/save`, `/load`

### 配置層級 (優先順序由高到低)
1. CLI 旗標 (--host, --port, --token)
2. 環境變數 (OLLAMA_HOST, OLLAMA_TOKEN)
3. 設定檔 (~/.ollamacli/config.yaml)
4. 預設值

### 錯誤碼設計
- `0`: 成功
- `1`: 一般錯誤
- `2`: 使用者輸入錯誤
- `3`: 無法連線至 Ollama server
- `4`: 認證失敗

## 開發流程

### 功能開發流程
1. 閱讀設計文件確認需求細節
2. 更新或建立對應的 ADR 文件
3. 設計 CLI 介面和內部 API
4. 實施核心業務邏輯
5. 撰寫單元測試和整合測試
6. 建立使用範例和文件
7. 執行 Makefile 測試流程確認功能正常
8. **自動觸發 progress-recorder** 記錄完成狀態

### 程式碼品質檢查
- 執行 gofmt 和 goimports
- 執行 golangci-lint
- 確保測試覆蓋率達標
- 檢查安全性漏洞

### 發佈檢查清單
- [ ] 所有測試通過 (unit, integration, e2e)
- [ ] Linting 工具無警告
- [ ] 版本號更新 (Semantic Versioning)
- [ ] CHANGELOG 更新
- [ ] 範例文件更新
- [ ] GoReleaser 配置正確
- [ ] 跨平台建置測試
- [ ] 使用手冊完整

## 專案特定配置

### CLI Application 相關
- 指令介面需保持向後相容性
- 輸出格式需考慮腳本化使用
- 錯誤訊息需清晰且可操作
- 支援 --help 和 man page
- 遵循 Unix philosophy (do one thing well)

### Ollama 整合
- 遵循 Ollama REST API 規格
- 支援串流回應處理
- 實施適當的連線重試機制
- 維護 API 相容性
- 提供離線模式支援 (本地模型)

## 關鍵提醒

⚠️ **重要**: 任何架構決策、技術選型、重要問題解決都必須立即使用 progress-recorder 記錄

🔄 **持續改進**: 定期使用 /recap 指令回顧專案狀態和待辦事項

📋 **文件更新**: 有任何重構或新增功能，請同步更新設計文件與相關文件

---
*此配置檔由 /claude-init 根據 PRD 檔案 docs/design.md 自動生成*
*生成時間: 2025-09-29 14:50:00*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/peihsinsu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/peihsinsu)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
