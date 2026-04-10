## autosr-sdse

> > 本檔同時供人類與開發代理（Codex、Cursor、Zed、Aider…）閱讀。請嚴格遵守。

# AGENTS.md

## 專案準則（請嚴格遵守）
---

> 本檔同時供人類與開發代理（Codex、Cursor、Zed、Aider…）閱讀。請嚴格遵守。  
> **重要：`sdse-uv/` 是專案的 Python 虛擬環境容器，不是專案名稱。**
> 請用中文回答，專有名詞請用原文。

---

- 回覆前必須先進行「事實檢查思考」，依序核對（a）使用者提供內容、（b）模型內部已知資訊、（c）至少一個可信的外部來源（網路搜尋、官方 repo、文件等）；除非使用者明確禁止或題目已提供完整證據，否則需主動執行外部查詢。
- 進行外部查詢時先簡述查詢動機與渠道，並於回覆中標示實際來源或檢核方式；屬於推論或假設的內容須清楚標註。
- 若外部查詢失敗或資料仍不足，須明確回覆「無法確定」或說明缺乏依據，禁止臆測、補完或擴大原意。
- 不得替使用者重寫或改述成不同語意；如需重述，請標示「重述版本」並保持語義對等。
- 回覆時避免使用「應該是」「可能是」「我猜」等模糊語氣，除非使用者明確要求。
- 產出前請自檢三項：a. 回覆有明確資料依據 b. 未超出問題範圍 c. 無新增未被提及的人名／數字／事件。

---

## 0) 預設與範圍
- 預設工作目錄：**倉庫根目錄**（含 `pyproject.toml`）。
- **除非我在對話中另有指定，所有指令一律在 `sdse-uv` 環境下執行。**
- 環境啟用：在每個新的 shell session 開始時先執行一次：  
  `source sdse-uv/.venv/bin/activate`  
  若該路徑不存在，改用：`source sdse-uv/bin/activate`。
- **不得**在此檔中擅自加入或執行「uv 安裝 / 依賴同步」等步驟（這些已記錄在其他檔案）；如需調整請開 PR 討論。

---

## 1) 專案結構（約定）
- 原始碼：`src/`
- 測試：`tests/`
- 文件：`docs/`
- 變更日誌：`logs/YYYY-MM-DD.md`
- 其它：`pyproject.toml`, `uv.lock` 置於倉庫根目錄

> 代理在自動新增檔案時，請依上列位置放置；不得將原始碼或文件寫入 `sdse-uv/`。

---

## 2) 標準指令（先啟用 sdse-uv 再執行）
- **執行主流程**（依實際入口替換）：  
  `python unified_research_pipeline.py --help`
- **單元測試**：  
  `pytest -q`
- **靜態檢查**：  
  `ruff check .`
- **自動格式化（如需要）**：  
  `ruff format .`
- **型別檢查（如已設定）**：  
  `mypy src/`

> 若工具不存在或配置缺失，請開 PR 補齊設定；**請勿**在未獲同意下自動安裝或變更依賴。

---

## 3) 環境變數與密鑰管理
- 機密資訊統一維護在倉庫根目錄的 `.env`（依 `.env.example` 建立），內容僅存放本地/臨時的 API key、token 等敏感資料。
- `.env` 已列入 `.gitignore`；嚴禁將任何密鑰、token、憑證寫入版本庫（含範例程式碼與文件）。
- 建議使用 `set -a; source .env; set +a` 將變數匯入當前 session，或以 `direnv` / 其他私密方案自動載入；勿在 shell 歷程中直接曝光密鑰。
- **程式碼要求**：任何需要 API key 或其他機敏環境變數的程式/測試入口檔案，必須在初始化階段載入 `.env`（可自行解析或使用既有工具如 `python-dotenv` 的 `load_dotenv()`）；不得假設操作者已手動匯入環境變數。
- `OPENAI_API_KEY` 已預先配置於 `.env`，所有會打到 OpenAI 的測試或流程都必須直接使用實際金鑰執行；若金鑰遺失或無效，應讓流程直接報錯而非以 skip/fallback 迴避，不需要確認這件事，直接跑就好了，有 error 再向我反應。
- 團隊共享密鑰時請透過可信任的秘密管理工具（Vault、1Password、Secrets Manager 等）或安全管道傳遞，並於共享文件中更新命名/使用約定。

---

## 4) 文件與日誌（強制）
- **文件**：每個功能或修正，都要新增/更新 `docs/<模組>/<功能>.md`，內容至少含：
  目的、介面/參數、使用範例、測試要點、已知限制。當次變更若僅涉及文件或日誌本身，可視為已符合此項要求。
- **日誌**：每完成一件事，於 `logs/YYYY-MM-DD.md` 追加一行：  
  `- [HH:MM ±TZ] [模組] [變更摘要] (#issue 或 PR 連結)`  
  其中時間請使用 24 小時制，並附上使用者所在時區（例：`+08`）。

---

## 5) Git 工作流與提交政策（嚴格）
- 允許直接 push 到遠端 default branch（目前為 `main`；以 `origin/HEAD` 為準）。
- 標準流程：**本地檢查（ruff/mypy/pytest）→ 更新 docs/logs → commit → push 到 default branch**；如需隔離工作可自行建立分支。
- **Commit 節奏**：一個邏輯單元一個 commit；訊息遵循 Conventional Commits（例：`feat(core): add scoring`）。
- **Agent 自動提交限制**：
  - 僅允許對 **`docs/**` 與 **`logs/**`** 的純內容變更自動 **commit & push 到 default branch**；
  - 任何涉及 **`src/**` 或 **`tests/**`** 的改動，**不得**自動提交，需由使用者確認後自行提交到 default branch。

---

## 6) 提交前自檢（必做）
- `ruff check .` ✅  
- `pytest -q` ✅  
- 若有啟用型別檢查：`mypy src/` ✅  
- 本次若改了 `src/**` 或 `tests/**` → **必須**同時更新 `docs/**` 或 `logs/**`（至少一者） ✅  

> CI 會比照上述條件執行；不合規變更將被退回修正。

---

## 7) spec-kit（後續導入）
本專案後續會使用 **spec-kit** 輔助開發；在導入完成前，代理請忽略 spec-kit 相關步驟。  
導入後將補充專用指令（示意）：
- 產生/同步規格：`spec-kit gen --in specs/ --out src/`
- 驗證規格符合度：`spec-kit check --in specs/ --target src/`

> 若指令不存在，請勿臆測執行；開 PR 討論再行導入。

---

## 8) 常見誤用（請避免）
- ❌ 將 `sdse-uv/` 視為專案根目錄或寫入任何專案檔案。
- ❌ 在未啟用 `sdse-uv` 環境下直接呼叫 `python/pytest/ruff`。
- ❌ 自動安裝依賴或變更環境設定（請改開 PR）。
- ❌ 未更新 `docs/` 或 `logs/` 就提交修改 `src/` 的 PR。
- ❌ 以 `pytest.skip`、`skipif`、`xfail`、mock/stub/recording、離線快照等手法繞過實際遠端服務；所有測試必須直接呼叫真實 API 並完整驗證行為。
- ❌ 在程式或測試中加入自動 fallback 或離線替代流程；若遠端服務失敗，必須直接回報錯誤，勿靜默容錯或改走模擬流程。
- ❌ 以 stub/Fake 方式自建假物件或假服務繞過真實流程；除非有充分理由且事前向使用者報備取得書面同意，否則一律禁止。
- ❌ 禁止將需因情境調整的輸入資料（例如使用者提示詞、查詢條件、憑證）硬編在程式中；這類內容必須透過環境變數、設定檔或資料來源注入。允許將公開、穩定的常數（如定價表、型號白名單）維護於原始碼，但需加註來源與更新時間。
---

## 9) 測試產出與暫存（SOP）
- `test_artifacts/`：保存需要人工檢視或後續分析的輸出；依測試套件、日期或任務建立子資料夾，必要時再挑選內容納入版本控管。
- `test/.tmp/`：僅供測試階段的暫存檔使用；測試腳本或清理指令（例：`make clean`）要自動刪除無用檔案。
- 所有測試流程在寫入前都必須 `mkdir -p test_artifacts test/.tmp`，避免在倉庫根目錄散落檔案。
- 若測試會下載/快取大檔，放在上述目錄並記錄來源；共享前請壓縮、移出倉庫或上傳至共用空間。
- 更新或清除 `test_artifacts/` 時請於日誌紀錄用途與狀態，避免舊資料混淆。

---

## 10) Docstring 與 AI 協作指引
- **公開 API 必備**：模組、類別與外部可呼叫的函式一律撰寫 docstring，說明用途、主要參數、回傳值與重要副作用，供人類與 AI 代理準確理解介面。
- **複雜內部邏輯**：若函式邏輯牽涉多步驟、狀態切換或關鍵假設，即使僅供內部使用也要加入精簡 docstring 或行內註解，避免 Copilot/LLM 產生錯誤建議。
- **避免噪音**：對於簡單且語意明確的小型 helper，可以仰賴清楚的命名與型別註解，毋須為了「每個函式都有 docstring」而強行重複內容。
- **維持一致風格**：撰寫時遵循既定 docstring 樣式（PEP 257/Google/NumPy 依專案既有慣例擇一），保持格式一致，讓 AI agent 更容易解析。
- **持續更新**：每次修改函式行為或參數簽名時，同步更新 docstring；若使用自動化工具（如 ChatGPT/Cursor）生成程式碼，請檢查並補強 docstring 的正確性與完整性。

---

## 11) 受保護檔案（請勿修改）
- 最新清單請參考 `docs/agent-protected-files.md`。
- 目前指定為 Codex 代理不得修改的檔案：
  - `src/utils/llm.py`
  - `src/utils/paper_downloaders.py`
  - `src/utils/paper_workflows.py`
  - `src/utils/env.py`
  - `src/utils/keyword_extractor.py`
  - `outputs/*/latex/figs/*.tex`（已生成的圖表程式碼，未經使用者明示禁止修改）
  - `outputs/*/tmp/mainbody_*.tex`（後處理主體內容，除非使用者要求，嚴禁覆寫或刪除）
- 若需調整清單內容，務必先取得使用者同意，再同步更新本節與 `docs/agent-protected-files.md`。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sologa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sologa)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
