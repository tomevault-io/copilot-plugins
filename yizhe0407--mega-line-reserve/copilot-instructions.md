## mega-line-reserve

> 後端採用 MVC + Repository 架構：

## 後端架構 (apps/api/src)

後端採用 MVC + Repository 架構：

- `controllers/` - 處理 HTTP 請求與回應，不寫業務邏輯
- `services/` - 業務邏輯層（權限檢查、驗證、流程控制）
- `models/` - 資料存取層（Prisma 操作）
- `routes/` - API 路由定義，只定義路由不寫邏輯
- `middleware/` - Express 中介軟體（auth、驗證、錯誤處理）
- `types/` - TypeScript 型別定義
- `config/` - 設定檔（資料庫、環境變數）
- `utils/` - 工具函式

## 前端架構 (apps/web/src)

- `app/` - Next.js App Router 頁面
- `components/` - React 元件（含 shadcn/ui）
- `hooks/` - 自訂 React Hooks
- `lib/api/` - API 呼叫函式
- `auth/` - 認證相關邏輯
- `config/` - 前端設定檔
- `constants/` - 常數定義
- `types/` - TypeScript 型別定義

## 開發注意事項

- 後端使用 ES Modules（package.json 有 "type": "module"）
- 前端使用 @tanstack/react-query 管理 API 狀態
- UI 元件基於 Radix UI + shadcn/ui
- 驗證使用 Zod schema
- 所有容器服務使用健康檢查確保穩定性
- 生產環境需要設定正確的環境變數（.env 檔案）

## 程式碼風格

- 程式碼註解一律使用繁體中文
- 回應與文件使用繁體中文
- 避免使用 Markdown 表格，改用清單呈現資訊

### 單一職責原則 (Single Responsibility Principle)

每個檔案只負責一件事，嚴禁混合不相關的邏輯：

- Controller：只處理 HTTP 請求/回應，不寫業務邏輯
- Service：只處理業務邏輯，不直接操作資料庫
- Model：只負責資料存取（Prisma 操作），不處理業務規則
- Route：只定義路由，不寫任何邏輯
- Component：一個元件只做一件事，複雜邏輯抽成 Hook 或 Utility
- Hook：只封裝可重用的狀態邏輯
- Utility：只放純函式，無副作用

## Git Commit 規範

使用 Conventional Commits + Gitmoji 規範：

### Commit Message 格式

```text
<emoji> <type>(<scope>): <subject>

<body>

<footer>
```

### Type 類型與對應 Gitmoji

- ✨ `feat`: 新功能
- 🐛 `fix`: 修復 bug
- 📝 `docs`: 文件變更
- 💄 `style`: 程式碼格式調整（不影響邏輯）
- ♻️ `refactor`: 重構（非新功能、非修復）
- ⚡️ `perf`: 效能優化
- ✅ `test`: 測試相關
- 🔧 `chore`: 建置流程或輔助工具變更
- 👷 `ci`: CI/CD 設定變更
- 🗃️ `db`: 資料庫相關變更
- 🔥 `remove`: 移除程式碼或檔案
- 🚀 `deploy`: 部署相關
- 🔒 `security`: 安全性修復
- ⬆️ `deps`: 升級依賴
- ⬇️ `deps`: 降級依賴
- 🎨 `format`: 改善程式碼結構/格式

### Scope 範圍（選填）

- `frontend`: 前端相關
- `backend`: 後端相關
- `db`: 資料庫相關
- `docker`: 容器相關

### 規則

- subject 必須使用 **英文**，需簡潔明瞭
- 必須搭配適合的 Gitmoji
- subject 不超過 50 字元
- body 用於說明「為什麼」做這個變更（選填，亦需使用 **英文**）
- 每個 commit 應該是一個獨立、完整的變更單位

### 範例

```text
✨ feat(api): Add user login API

🐛 fix(web): Fix course list pagination issue

📝 docs: Update README installation instructions

♻️ refactor(api): Refactor auth middleware

⬆️ chore: Update pnpm dependency versions

🗃️ db: Add course table migration
```

## 最佳實踐思考流程

在進行任何實作之前和之後，都需要進行最佳實踐的思考。

### 實作前思考

在開始寫程式碼之前，先問自己：

- 這個解法是否符合專案的架構模式？
- 是否有現成的工具/函式庫可以使用？
- 這樣做是否符合單一職責原則？
- 是否會造成重複程式碼？能否抽象化？
- 效能考量：這個做法在大量資料下是否有問題？
- 安全性考量：是否有潛在的安全漏洞？
- 可維護性：未來其他人能否輕易理解這段程式碼？

### 實作後反思

完成實作後，回頭檢視：

- 程式碼是否簡潔易讀？
- 是否有更好的替代方案？
- 錯誤處理是否完善？
- 邊界情況是否都有考慮到？
- 是否需要加入註解說明複雜邏輯？
- 是否符合專案既有的程式碼風格？

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yizhe0407)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Yizhe0407)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
