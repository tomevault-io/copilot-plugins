## cleanarchitecture-northwind

> 本專案以 `jasontaylordev/CleanArchitecture` 為基礎，採用 Clean Architecture、CQRS、MediatR、EF Core 與 ASP.NET Core MVC。

# AGENTS.md

## 1. 專案定位與核心原則

本專案以 `jasontaylordev/CleanArchitecture` 為基礎，採用 Clean Architecture、CQRS、MediatR、EF Core 與 ASP.NET Core MVC。

| 項目 | 說明 |
| --- | --- |
| Presentation | `src/Web` 使用 ASP.NET Core MVC |
| Core | `src/Application` 與 `src/Domain` 是系統核心 |
| Infrastructure | `src/Infrastructure` 負責 EF Core、Identity 與外部服務實作 |
| Shared | `src/Shared` 僅放跨專案共用且不含業務規則的內容 |

核心原則：

1. `Domain` 不依賴任何外層。
2. `Application` 只依賴 `Domain`，不得依賴 `Infrastructure` 或 `Web`。
3. `Infrastructure` 實作 `Application` 定義的抽象。
4. `Web` 是 MVC Presentation；業務流程透過 `Application`，不得直接操作資料庫或 Infrastructure 實作。
5. `Web` 可在 composition root 引用 Infrastructure 的 DI 註冊擴充，但 Controller、ViewModel 與 View 不得直接依賴 Infrastructure 實作。
6. 實際 Repository 結構與既有慣例優先於本文件中的示意路徑，不得虛構檔案或建立平行架構。
7. 本專案目前不使用 .NET Aspire；除非使用者明確要求評估或重新導入，否則不得新增相關專案、套件或 hosting 設定。

所有變更必須優先維持架構邊界，不得為了快速完成而讓 MVC、EF Core、外部服務或基礎設施細節滲入核心層。

## 2. 回覆與變更原則

### 2.1 語言與格式

Codex 回覆必須使用繁體中文。

回覆應依任務複雜度選擇格式：

| 類型 | 要求 |
| --- | --- |
| 簡單說明 | 直接回答，先提供結論或下一步 |
| 技術分析 | 只有比較方案、矩陣資料或結構化結果時才使用表格 |
| 實作建議 | 只有涉及多步驟或使用者要求時才使用 SOP |
| 程式碼 | 使用正確語言標記，只展示與任務相關的必要片段 |
| 架構說明 | 只有涉及架構、跨層流程或依賴方向時才展開 |
| 完成回報 | 優先回報變更、實際驗證與必要風險 |

避免重複完整計畫、已知背景、未受影響的架構規則與空白報告區段。

### 2.2 變更限制

| 禁止項目 | 說明 |
| --- | --- |
| 虛構內容 | 不得假設不存在的檔案、類別、方法、設定或測試專案 |
| 大範圍重構 | 除非任務明確要求 |
| Hacky 解法 | 不得以繞過架構、安全或驗證方式完成 |
| 無關修改 | 不得修改與任務無關的格式、命名、套件或設定 |
| 未經要求新增套件 | 不得任意新增 NuGet package |
| 未經要求修改部署 | 不得任意修改 Docker、CI/CD 或環境設定 |
| 未經要求導入框架 | 不得新增會改變既有架構、hosting 或部署模式的框架 |

採取最小、可維護、可驗證的修改；找到足以安全完成任務的證據後停止擴大探索。

## 3. 專案結構

目前主要結構：

```text
CONTEXT.md

docs/
  adr/
    0001-*.md

src/
  Application/
  Domain/
  Infrastructure/
  Shared/
  Web/

tests/
  Application.FunctionalTests/
  Application.UnitTests/
  Domain.UnitTests/
  Infrastructure.IntegrationTests/
```

文件責任：

| 文件 | 唯一責任 |
| --- | --- |
| `AGENTS.md` | Agent 的工作方式、架構限制、安全規則與驗證要求 |
| `CONTEXT.md` | Northwind 領域詞彙表；只定義業務概念與統一用語 |
| `docs/adr/*.md` | 難以回復、具有真實取捨的重要架構決策及其原因 |

若實際結構與上述不同，必須以實際檔案為準。

## 4. 分層責任與依賴方向

本節是分層責任、允許依賴與禁止依賴的唯一規範來源；後續章節只補充該層的實作特例。

### 4.1 分層責任

| 專案 | 角色 | 可放內容 | 禁止內容 |
| --- | --- | --- | --- |
| `Domain` | 核心領域模型 | Entity、Value Object、Domain Event、Enumeration、Domain Rule | EF Core、HTTP、MVC、MediatR Handler、外部 API、資料庫細節 |
| `Application` | Use Case 與流程協調 | Command、Query、Handler、DTO、Validator、Interface、Pipeline Behaviour | Controller、ViewModel、DbContext 實作、Razor、HttpContext、外部服務實作 |
| `Infrastructure` | 技術實作 | EF Core、Identity、Repository 實作、Email、File Storage、第三方 API Client | MVC、ViewModel、Use Case 流程、業務規則 |
| `Web` | ASP.NET Core MVC Presentation | Controller、View、ViewModel、Filter、Middleware、路由、DI 組合、UI 驗證 | 商業邏輯、直接操作 DbContext、直接呼叫 Infrastructure 實作、直接暴露 Domain Entity |
| `Shared` | 無業務規則的跨專案內容 | 共用常數、共用契約、共用基底型別 | 特定 Use Case、UI 文字、資料存取、業務規則 |

### 4.2 允許依賴

| From | Allowed To |
| --- | --- |
| `Domain` | 無 |
| `Application` | `Domain` |
| `Infrastructure` | `Application`、`Domain`、必要時 `Shared` |
| `Web` | `Application`、composition root 所需的 `Infrastructure`、必要時 `Shared` |
| `Shared` | 無業務層依賴 |

### 4.3 禁止依賴

| 禁止方向 | 原因 |
| --- | --- |
| `Domain` -> 外層 | Domain 必須是最內層 |
| `Application` -> `Infrastructure` | Use Case 只能依賴抽象 |
| `Application` -> `Web` | Use Case 不得依賴 MVC |
| `Infrastructure` -> `Web` | 技術實作不得依賴 Presentation |
| Controller -> EF Core `DbContext` | Web 不得直接存取資料庫 |
| Controller -> Infrastructure 實作 | 業務流程應透過 Application；僅 composition root 可註冊實作 |
| View -> Domain Entity | View 必須使用 ViewModel |

### 4.4 各層實作規則

**Web／MVC**

- Controller 接收 HTTP 輸入、處理 ModelState 與導航，將 ViewModel 映射為 Command／Query，透過 `ISender` 呼叫 Application，再將 DTO 映射為 ViewModel。
- Controller 不得使用 Service Locator 處理業務流程，且必須傳遞 `CancellationToken`。
- ViewModel 僅用於顯示與 UI 驗證，不得包含業務規則或包裝 EF Core tracking entity；Application DTO 不得依賴 MVC。
- View 只負責顯示、表單輸入、基本 UI 判斷、Tag Helper 與 ModelState 錯誤呈現，不得查詢或修改資料。
- Cookie 驗證的瀏覽器狀態變更 Request 必須使用 Anti-Forgery；Bearer Token API、Webhook 與第三方 callback 依其驗證模型處理。

**Application 與 Domain**

- Application 是 Use Case 中心；不得出現 `Controller`、`IActionResult`、`ViewResult`、`HttpContext`、`TempData`、Razor、HTML、DbContext 實作或外部 API SDK 實作。
- Domain 錯誤使用穩定代碼或語意型例外，不直接寫入中文 UI 訊息。

**Infrastructure 與 EF Core**

- 查詢優先 projection；read-only 查詢優先 `AsNoTracking()`；避免 N+1、過早 `.ToList()` 與迴圈逐筆查詢。
- Raw SQL 必須參數化；非同步查詢、儲存與外部呼叫必須傳遞 `CancellationToken`。
- Migration 僅在任務明確要求或模型變更必要時新增。

**Web composition root 與 DI**

- Application 與 Infrastructure 分別提供 DI 註冊擴充方法，僅由 Web composition root 組合；Options 在此綁定，Secret 不得硬編碼。
- Controller 優先注入 `ISender` 或 Web concern service，不得手動建立 Infrastructure 實作。
- Health Check、Telemetry 與路由設定沿用既有架構；MVC 使用 `AddControllersWithViews()` 與既有 Controller route，API Controller 存在時可同時 `MapControllers()`。

### 4.5 Mapping 規則

| From | To | 位置 |
| --- | --- | --- |
| MVC ViewModel | Command | Controller 或 Web mapping |
| Query DTO | MVC ViewModel | Controller 或 Web mapping |
| Domain Entity | Application DTO | Application |
| EF Core Projection | Application DTO | Application Query 或 Infrastructure 實作 |
| Domain Entity | MVC ViewModel | 不得直接 mapping，應經 Application DTO |

1. 大量同名欄位、巢狀物件或集合轉換時，優先使用既有 `MapsterMapper.IMapper`。
2. 少量欄位、UI 格式化、遮罩或明確判斷，可在 Web 或 Handler 顯式撰寫。
3. Application 只設定 Domain、DTO、Command 等可見型別的映射；DTO 到 MVC ViewModel 的組態放在 Web。
4. EF Core 查詢只有在 `ProjectToType` 可正確轉譯且不造成 N+1 時才使用。
5. 不得為單一需求引入新的 mapping 套件。

### 4.6 典型資料流

Command：`Browser -> Web Controller／ViewModel -> ISender.Send(Command) -> Application Handler -> Domain -> Application Interface -> Infrastructure -> Database／External System -> Redirect／View`。

Query：`Browser -> Web Controller -> ISender.Send(Query) -> Application Handler -> Application Data Abstraction -> Infrastructure Projection -> Application DTO -> Web ViewModel -> Razor View`。Query 不一定需要載入 Domain Entity，應優先使用可翻譯 projection。

### 4.7 常見需求放置位置

| 需求 | 正確位置 |
| --- | --- |
| MVC 頁面、UI 文字 | `Web/Controllers`、`Web/Views`、`Web/ViewModels` |
| 查詢功能 | Application Query + Handler |
| 寫入功能 | Application Command + Handler + Validator |
| 核心業務 Entity | `Domain` |
| EF Core 設定 | `Infrastructure` |
| Email 或檔案儲存 | Application 介面、Infrastructure 實作 |
| Health Check | `Web` 或 `Infrastructure` |
| 測試資源 | 實際存在且責任相符的測試專案 |

## 5. CQRS、MediatR 與驗證

### 5.1 Command 與 Query

- Command 用於改變狀態，常用命名為 `CreateXxxCommand`、`UpdateXxxCommand`、`DeleteXxxCommand`、`ChangeXxxStatusCommand`。
- Query 用於讀取資料且不得修改狀態，常用命名為 `GetXxxsQuery`、`GetXxxDetailQuery`、`GetXxxOptionsQuery`、`SearchXxxsQuery`。

### 5.2 Handler

1. Handler 放在 `Application` 並協調 Use Case。
2. Handler 可呼叫 Application 定義的抽象，不得依賴 Infrastructure 實作。
3. Handler 不得知道 MVC、Controller、ViewModel、TempData、HTML 或 Razor。
4. Handler 不得直接讀取 `HttpContext`；目前使用者資訊應透過 Application 抽象，例如 `IUser` 或既有介面。
5. 非同步 Handler、資料存取與外部服務呼叫應一路傳遞 `CancellationToken`。

### 5.3 Validator

| 驗證類型 | 放置位置 |
| --- | --- |
| Use Case 輸入規則 | `Application` Validator |
| UI 顯示格式或必填提示 | MVC ViewModel |
| 不變的業務規則 | `Domain` |

不可只在 MVC ViewModel 驗證業務規則；Application 必須保護 Use Case 邊界。

## 6. 安全規則

| 風險 | 規則 |
| --- | --- |
| CSRF | 使用 Cookie 驗證的瀏覽器狀態變更 MVC Request 必須使用 Anti-Forgery |
| Overposting | 不得直接 bind Domain Entity 或 EF Entity |
| XSS | 使用 Razor 預設編碼，不得任意使用 `Html.Raw` |
| 機敏資料 | 不得輸出 password、token、secret、connection string 或未遮罩個資 |
| Logging | 不得在 Log、Exception、Telemetry 或追蹤資訊中記錄機敏資料、Data Protection 解密結果或完整受保護字串 |
| 錯誤訊息 | 不得把 exception detail 直接顯示給使用者 |
| 授權 | 需要限制的頁面或操作必須使用 `[Authorize]`、Policy 及資源層級授權 |
| 檔案上傳 | 驗證副檔名、大小、Content-Type、儲存路徑與檔名 |
| Redirect | 避免 open redirect；外部 URL 必須驗證 |
| ModelState | 驗證失敗必須以既有 UI 流程安全回應 |

### 6.1 Controller 與 View 間的敏感資料保護

僅在內部識別值、可被竄改而改變操作的值，或需限定 purpose／有效期限時保護；不要因欄位名為 `Id` 就一律保護。受保護字串仍可能出現在 HTML、URL、Log 或 Proxy，並不取代授權。

1. **邊界**：Data Protection 屬於 Web concern。Controller 將必要原始值保護後放入 ViewModel；接收時先還原、驗證，再建立只含業務型別的 Command 或 Query。`Application` 與 `Domain` 不得依賴 `IDataProtector` 或受保護字串格式。
2. **實作**：purpose 必須穩定且限縮功能範圍，例如 `TodoItems.Edit.ItemId.v1`；ViewModel 使用 `ProtectedId` 等欄位名；只保護必要欄位。多處共用時在 Web 建立封裝。
3. **失敗與安全**：還原失敗、空值、格式錯誤或竄改時中止操作且不洩漏例外。仍須執行授權、資源擁有權、Anti-Forgery 與業務驗證。高敏感資料避免 URL；時效連結使用 time-limited protector，一次性操作另加伺服器端重放防護。
4. **部署與測試**：Production 持久化並共用 key ring 與 Application Name。至少驗證合法 round-trip、竄改或錯誤 purpose、還原失敗不送出 Command／Query、未授權拒絕，以及適用時的逾期與重放。

## 7. 測試與驗證

### 7.1 測試放置

| 測試類型 | 專案 |
| --- | --- |
| Domain 規則 | `tests/Domain.UnitTests` |
| Application Handler | `tests/Application.UnitTests` |
| Application 流程與資料庫整合 | `tests/Application.FunctionalTests` |
| Infrastructure 實作 | `tests/Infrastructure.IntegrationTests` |
| MVC Controller、路由或 View 行為 | 使用 Repository 內實際存在且責任最接近的測試專案 |

### 7.2 驗證範圍

| 變更類型 | 最低建議驗證 |
| --- | --- |
| Markdown、README、註解或純文字 | 檢查格式、連結與內容；通常不需 build/test |
| 單一方法或單一層局部修改 | 受影響專案 build 與最相關測試 |
| 跨層流程、公開介面或共用元件 | 相關專案 build、單元測試與整合測試 |
| 套件、啟動、DI、資料模型或 schema | 完整 `dotnet build` 與相關完整測試 |
| 合併前、Pull Request 或明確要求 | 完整 `dotnet build` 與 `dotnet test` |

常用命令：

```bash
dotnet build
dotnet test
dotnet test tests/Domain.UnitTests
dotnet test tests/Application.UnitTests
dotnet test tests/Application.FunctionalTests
dotnet test tests/Infrastructure.IntegrationTests
```

只有在本次工作中實際執行並確認輸出成功，才可宣稱 build、test、lint 或修正通過。

若必要驗證無法執行，回覆中必須列出未執行命令、原因、未驗證風險與可手動執行的命令。

## 8. CodeGraph 使用規則

CodeGraph 用於理解跨檔案、跨類別與跨分層關係。它是架構探索與影響分析工具，不取代實際讀檔、Git、建置或測試。

### 8.1 優先使用時機

| 場景 | 目的 |
| --- | --- |
| 架構與模組關係 | 理解 Web、Application、Domain、Infrastructure 與 Shared 的互動 |
| HTTP 或功能流程 | 追蹤 Controller、Command／Query、Handler、Domain 與 Infrastructure |
| 呼叫鏈與符號關聯 | 找出呼叫端、被呼叫端、介面與實作 |
| 跨層修改 | 確認 MVC、Use Case、Domain、EF Core 或外部服務影響 |
| 公開符號修改 | 評估刪除、重新命名或介面更換的影響範圍 |
| 不熟悉模組 | 快速找出入口、核心符號與相關檔案 |
| 程式碼審查 | 補充相依性與遺漏影響分析 |
| 架構違規調查 | 尋找不合法依賴或資料流 |

使用原則：

1. 優先使用 `codegraph_explore` 找出符號、檔案、呼叫鏈與相依關係。
2. 找到目標後，修改前仍須直接讀取目前工作樹中的檔案。
3. 結果不足時再使用 `rg`、檔案搜尋、Git 或直接讀檔補充。
4. 索引過期時先更新或說明限制。
5. 只有實際呼叫 CodeGraph 才可聲稱已使用。

### 8.2 通常不需要使用的場景

- 使用者已指定明確檔案與修改位置。
- README、Markdown、註解或文字修正。
- 單一設定檔檢查。
- Build、test、format 或靜態分析。
- 已有明確錯誤訊息、stack trace 或失敗測試。
- 套件版本與弱點檢查；除非需分析受影響呼叫範圍。
- 單一方法內部或局部靜態資源調整。
- Git 狀態、diff、commit 範圍與變更清單。

### 8.3 不可用或資料不足時

1. 簡要說明原因。
2. 改用 `rg`、Git、檔案搜尋與直接讀檔完成可安全處理的工作。
3. 不得因 CodeGraph 不可用而停止整個任務。
4. 不得假裝已使用或虛構符號、檔案與呼叫關係。

## 9. Codex 實作 SOP

1. 先讀取使用者指定檔案，以及與任務直接相關的實作、參照與測試。
2. 若任務涉及 Northwind 業務概念、命名或領域邊界，先讀取 `CONTEXT.md`；若涉及既有架構選擇，先讀取 `docs/adr/` 中相關 ADR。
3. 依第 8 節判斷是否需要 CodeGraph；只有跨層、呼叫鏈、公開介面或影響分析時才優先使用。
4. 檢查既有命名、資料夾、實作與測試風格，避免建立平行模式。
5. 判斷功能屬於 Domain、Application、Infrastructure、Web、Shared 或測試專案。
6. 只修改任務涉及的層；不得為了流程完整而建立不需要的檔案。
7. 補上與變更風險相稱的測試。
8. 依第 7 節執行最小且足以證明結果的驗證。
9. 精簡回報變更、實際驗證與必要風險。

若程式碼、使用者說法、`CONTEXT.md` 或 ADR 互相矛盾，必須先指出衝突並確認哪一項是新的來源事實，不得默默選擇其中一方。

建議實作順序：Domain 規則 → Application Use Case → Infrastructure 實作 → Web UI；只執行任務實際涉及的步驟。

## 10. 禁止修改區域與產生檔案

除非任務明確要求，不得修改：

| 區域 | 原因 |
| --- | --- |
| `Directory.Packages.props` | 影響全域套件版本 |
| `global.json` | 影響 SDK |
| Dockerfile / compose | 影響部署 |
| CI/CD workflow | 影響建置流程 |
| `appsettings*.json` | 可能影響環境設定 |
| Migration | 改變資料庫 schema |
| `.editorconfig` | 影響全專案格式 |

不得直接修改 `bin/`、`obj/`、測試結果、編譯輸出、產生的 OpenAPI client 或其他工具自動產生檔案；應修改其來源、模板或產生設定後重新產生。

### 10.1 Git 忽略與提交檢查

`.gitignore` 只會排除尚未追蹤的檔案；不會自動停止追蹤已暫存或已提交的檔案。

1. 不得使用 `git add -f` 或其他強制加入方式提交被 `.gitignore` 排除的本機執行狀態、工作報告或工具產物，例如 `.superpowers/`。
2. 建立提交前，必須檢查暫存內容；至少執行 `git diff --cached --name-only`，確認其中沒有 `.superpowers/`、`bin/`、`obj/`、測試結果或其他應忽略的路徑。
3. 若發現已追蹤的檔案後來被加入 `.gitignore`，必須先以 `git rm --cached -- <path>` 停止追蹤（保留本機檔案），再建立移除追蹤的提交。
4. 若確有必要版本控制被忽略路徑中的特定檔案，必須取得使用者明確要求、以最小範圍設定例外規則，並在提交回報中說明原因。

## 11. 任務完成回覆

### 11.1 小型或局部任務

只需回報：

1. 修改內容或結論。
2. 實際執行的驗證；未執行時說明原因。
3. 必要的未驗證事項或風險。

不存在的項目不得輸出空標題。

### 11.2 跨檔案、跨層或 Pull Request

依實際內容使用下列區段，未涉及者省略：

```markdown
## 變更摘要

| 類型 | 檔案 | 說明 |
| --- | --- | --- |
| ... | `...` | ... |

## 架構判斷

只有涉及架構或跨層變更時說明。

## 測試結果

| 命令 | 結果 |
| --- | --- |
| `...` | 通過、失敗或未執行及原因 |

## 風險與後續建議

只列出尚未驗證、可能影響或必要補測項目。
```

## 12. Skills、領域文件與輸出模式

本節是對已安裝 Skills 自動觸發規則的專案級覆寫。使用者要求、安全規則與本文件優先於 Skill。

`AGENTS.md` 主要決定是否啟用 Skill；啟用後依該 Skill 當前的 `SKILL.md` 執行。若 Skill 與本文件衝突，以使用者明確要求、安全規則與本文件為準。

不得僅因 Skill 已安裝或存在極低的適用可能性，就套用完整流程。

| Skill | 啟用條件 | 不應自動啟用的情況 |
| --- | --- | --- |
| `grill-me` | 使用者明確輸入 `/grill-me`、`$grill-me`，或明確要求逐題盤問、壓力測試計畫或設計，但未要求同步維護領域文件 | 一般需求略有不明確、存在多個方案、純程式碼說明或已確認可直接執行的局部任務 |
| `grill-with-docs` | 使用者明確輸入 `/grill-with-docs`、`$grill-with-docs`，或明確要求針對計畫逐題盤問、壓力測試並同步維護領域文件 | 一般需求略有不明確、存在多個方案、純程式碼說明或已確認可直接執行的局部任務 |
| `systematic-debugging` | 根因不明、修正曾失敗、測試反覆失敗或現象與預期不一致 | 已有明確根因的局部修正 |
| `test-driven-development` | 新增或修改明確可測且具中高風險的行為 | 文件、設定文字或極小型低風險修改 |
| `verification-before-completion` | 準備宣稱程式碼、設定、build 或 test 已完成或通過 | 不涉及完成狀態聲明的純說明 |
| `requesting-code-review` | 完成重大功能、複雜修正後或合併前，需要獨立 reviewer | 小型文件或局部低風險修改 |
| `writing-plans` | 多專案、多階段、公開介面、schema 或高風險變更 | 已有完整步驟的簡單任務 |
| `i-have-adhd` | 使用者明確輸入 `$i-have-adhd`，或明確要求後續回答保持簡短、行動導向 | 不得設為所有 Session 的全域預設 |

### 12.1 `grill-me` 執行規則

`grill-me` 僅用於釐清及壓力測試計畫或設計；除非使用者另行要求，不得更新 `CONTEXT.md`、建立 ADR 或開始實作。

### 12.2 `grill-with-docs` 執行規則

啟用後必須遵守：

1. 先探索 Repository、`CONTEXT.md` 與相關 ADR；可從程式碼或文件取得的事實不得詢問使用者。
2. 對尚未決定的設計分支，一次只提出一個問題，等待使用者回答後再繼續。
3. 每個問題必須附上 Codex 的建議答案與理由，不得只把決策負擔丟給使用者。
4. 使用具體情境與邊界案例壓力測試設計，並檢查使用者描述是否與程式碼、詞彙表或既有 ADR 衝突。
5. 領域詞彙一旦確認，立即更新 `CONTEXT.md`，不得等到整場盤問結束後才批次補寫。
6. 只有符合第 12.4 節門檻的決策才建立 ADR，不得為每個技術選擇建立文件。
7. 直到使用者明確確認「已達成共同理解」前，不得開始實作、修改程式碼、建立 Migration 或改動設定。
8. Grilling 結束後先整理共同理解、更新的領域詞彙與 ADR；只有使用者另行要求實作時，才進入計畫與實作流程。

### 12.3 `CONTEXT.md` 規則

本 Repository 採單一 Context，領域詞彙表位於根目錄 `CONTEXT.md`。

`CONTEXT.md` 必須：

- 只定義 Northwind 特有的業務概念與標準用語。
- 每個詞彙以一至兩句說明「它是什麼」。
- 一個概念只選一個 canonical term；容易混用的名稱列於 `_Avoid_`。
- 明確區分 `User`、`Employee`、`Customer` 等容易混淆的概念。
- 在詞彙與程式碼或使用者說法衝突時，立即指出並要求釐清。

`CONTEXT.md` 不得包含：

- ASP.NET Core、EF Core、MediatR、Mapster、Data Protection 等實作技術。
- 專案分層、資料夾結構、套件版本、建置或部署方式。
- 功能規格、驗收條件、待辦事項、會議筆記或設計草稿。
- 架構決策與取捨理由；這些內容應放在 ADR。

### 12.4 ADR 規則

ADR 的 canonical 位置為 `docs/adr/`，檔名使用依序編號：

```text
docs/adr/0001-short-decision-title.md
docs/adr/0002-another-decision.md
```

只有同時符合下列三項條件才建立 ADR：

1. **Hard to reverse**：日後改變會有明顯的程式、資料、部署、相容性或團隊遷移成本。
2. **Surprising without context**：未記錄原因時，未來維護者很可能質疑為何如此選擇。
3. **Real trade-off**：確實比較過可行替代方案，並因特定理由選擇其中之一。

ADR 應以短篇幅記錄決策、背景與原因；只有真正增加理解時才加入狀態、替代方案或後果。決策改變時新增 ADR 並標記取代關係，不得直接改寫歷史原因，使舊決策看起來從未存在。

### 12.5 其他 Skill 與輸出模式

啟用 `i-have-adhd` 時，先提供答案或下一步、避免重複與岔題；但不得省略安全風險、架構違規、測試失敗、未驗證事項或破壞性操作警告。輸入 `stop adhd mode` 或 `normal mode` 後停止套用。

下列任務通常直接處理，不使用額外 Skill：

- Markdown、README、註解與文字調整。
- 明確且局部的低風險程式修改。
- 查詢符號、檔案或解釋現有行為。
- 單一設定值檢查。
- 明確指定的 build、test 或 Git 命令。
- 已有完整步驟且不需重新設計的任務。

---
> Source: [soutsjjw/CleanArchitecture.Northwind](https://github.com/soutsjjw/CleanArchitecture.Northwind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
