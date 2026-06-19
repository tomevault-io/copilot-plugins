## medical-literature-review

> 撰寫醫學文獻回顧（literature review）時使用。執行「三層外部知識管線 + 第二輪 PDF 深挖」流程：外部搜尋 → 全文落地 → 程式化比對，並在第一輪 PDF 閱讀完後追加第二輪參考文獻搜尋與下載。LLM 僅做改寫，所有事實必須能被 grep 驗回外部來源。觸發詞：「文獻回顧」「literature review」「整理 paper」「寫 review article」「醫學寫作」「PubMed / Europe PMC / OpenEvidence / Consensus 整合」。


# 醫學文獻回顧：三層外部知識管線 + 第二輪深挖

> **v2.0 變更摘要**（2026-04）：`claude.ai PubMed` connector 已退役，改用 `paper-search-mcp`（<https://github.com/openags/paper-search-mcp>）作為第一層搜尋與第二層下載的統一前端。新工具支援 21 個來源（arXiv / PubMed / bioRxiv / medRxiv / PMC / Europe PMC / Semantic Scholar / Crossref / OpenAlex / CORE / OpenAIRE / dblp / DOAJ / BASE / Zenodo / HAL / SSRN / IACR / CiteSeerX / Unpaywall / 可選 Sci-Hub），並內建 **`download_with_fallback`** 的 OA-first 下載鏈（source-native → OpenAIRE/CORE/Europe PMC/PMC → Unpaywall DOI → 可選 Sci-Hub）。

## 核心原則：**不信任 LLM 的內部知識**

LLM 對醫學細節（作者、期刊、年份、DOI、數字、樣本數）會產生**看似合理但錯誤的輸出**——這就是幻覺。臨床寫作一旦幻覺，代價是決策失誤。本流程從頭到尾守一條硬規則：

> **LLM 只做組織、改寫、語氣處理；所有事實性內容必須有外部來源可追溯，且能被 `grep` 驗回本地全文。**

LLM 不是知識來源，而是編輯。真相來自 paper-search-mcp 查到的 metadata、OpenEvidence / Consensus 的 citation、以及**本地 PDF→Markdown** 後可以 grep 的全文。

---

## 流程總覽

```
[1] 第一層：外部搜尋與真實性驗證（paper-search-mcp 多源並行）
        ↓
[2] 第二層：全文取得（download_with_fallback → LlamaParse → 本地 MD）
        ↓
[3] ★ 第二輪深挖：讀完 PDF → 抓內文 references → 重做第一層 + 第二層
        ↓
[4] 第三層：幻覺校正（DOI / metadata / 數字 grep）
        ↓
[5] 交稿：標記 📄（本地全文驗證）/ 📌（僅 abstract）
```

---

## 第一層｜來源搜尋與真實性驗證（不下載全文，先篩）

**主力工具：`paper-search-mcp`** — 一次呼叫覆蓋 21 個公開來源，內建去重。

### 1.1 Tier-A：並行多源搜尋（預設）

| 工具 | 角色 |
|---|---|
| **`mcp__paper-search__search_papers`** | Layer-1 高階工具：多源並行搜尋 + 去重；預設同時打 arXiv / PubMed / bioRxiv / medRxiv / Semantic Scholar / Crossref / OpenAlex / Europe PMC / PMC 等。**這是每個 topic 開工的第一刀**。 |

### 1.2 Tier-B：來源專屬 Search（針對性補強）

| 工具 | 用途 |
|---|---|
| **`search_pubmed`** | PubMed：最權威生醫索引；確認論文真實存在、作者/期刊/年份正確 |
| **`search_europepmc`** | Europe PMC：PubMed 的歐洲鏡像 + 更多 preprint / 灰色文獻 |
| **`search_pmc`** | PubMed Central：全文 OA 生醫論文 |
| **`search_semantic`** | Semantic Scholar：引用圖譜、influential citation 標記 |
| **`search_crossref`** | Crossref：DOI metadata 最權威來源 |
| **`search_openalex`** | OpenAlex：開放學術知識圖譜；institution / author ID 追蹤 |
| **`search_biorxiv`** / **`search_medrxiv`** | 生醫預印本，補 PubMed 尚未收錄的新研究 |
| **`search_google_scholar`** | bot-detection 活躍，需設 `PAPER_SEARCH_MCP_GOOGLE_SCHOLAR_PROXY_URL`；作為最後補網用 |
| **`search_unpaywall`** | DOI-centric OA metadata（**需設 `PAPER_SEARCH_MCP_UNPAYWALL_EMAIL`**） |
| **`search_core`** | CORE：全球 repository 聚合（建議設 `PAPER_SEARCH_MCP_CORE_API_KEY`） |
| **`search_openaire`** / **`search_dblp`** / **`search_doaj`** / **`search_zenodo`** / **`search_hal`** / **`search_ssrn`** / **`search_base`** / **`search_citeseerx`** | 依 topic 需要選擇性補網 |

### 1.3 臨床問答 + 立場統整（paper-search-mcp 不涵蓋）

| 工具 | 角色 |
|---|---|
| **`mcp__openevidence__oe_ask`** | 臨床問答專家系統，回傳必附 citation；快速建立觀點地圖 |
| **`mcp__claude_ai_Consensus__`** | 跨論文立場統整（贊成/反對某結論的研究比例） |
| **`mcp__tavily__tavily_search / tavily_research`** | 廣域 web 搜尋（guideline、學會聲明） |

**這層輸出**：一份「值得納入引用」的論文清單（含 PMID / DOI）。LLM 之後只能在這份清單內作文，**不得自創引用**。

### 1.4 論文真實性驗證（取代舊 `lookup_article_by_citation`）

收到 LLM 給的候選引用後（含 LLM 可能幻覺的情況），用以下順序驗：

1. 若有 **PMID** → `search_pubmed(query="<PMID>")` 或 `search_pubmed(query="<title>")` 比對 title/author/year
2. 若有 **DOI** → `search_crossref(query="<DOI>")` 取 metadata；或 `search_unpaywall(query="<DOI>")` 看 OA 狀態
3. 若都無 → `search_papers(query="<title> <first_author> <year>")` 打多源並行，看哪個來源回傳並比對 metadata
4. 三路皆無 → **判為 fabricated citation，剔除**

---

## 第二層｜全文取得（拿到 PDF → 轉 Markdown → 本地可 Grep）

### 2.1 主力：`download_with_fallback`（paper-search-mcp 內建 OA-first 鏈）

| 工具 | 行為 |
|---|---|
| **`mcp__paper-search__download_with_fallback`** | 按順序嘗試：**source-native download → OpenAIRE/CORE/Europe PMC/PMC 發現 → Unpaywall DOI 解析 → 可選 Sci-Hub**。一次呼叫自動走完 OA-first fallback chain。 |

這取代了舊版的 `research_hub → Europe PMC 直連 → scihub-mcp` 手動三段式。

### 2.2 來源專屬 download / read（某些來源內建 full-text 讀取）

| 工具 | 備註 |
|---|---|
| **`download_arxiv` / `read_arxiv_paper`** | arXiv OA 全文（`read_*` 直接回傳文字，可跳過 LlamaParse） |
| **`download_biorxiv` / `read_biorxiv_paper`** | bioRxiv |
| **`download_medrxiv` / `read_medrxiv_paper`** | medRxiv |
| **`download_semantic` / `read_semantic_paper`** | Semantic Scholar（只對 OA 有效）|
| **PMC / Europe PMC `download_*`** | OA PDF 下載 |
| **`download_iacr` / `read_iacr_paper`** | 密碼學預印本 |
| **`download_scihub`** | paper-search-mcp 內建 scihub fallback（與 `mcp__scihub__` 等價，使用者自負法律責任）|

### 2.3 備援：獨立 scihub-mcp（當 paper-search-mcp 的 scihub fallback 失敗時）

| 工具 | 備註 |
|---|---|
| **`mcp__scihub__find_pdf` / `download_pdf`** | Playwright bypass DDoS-Guard 的本地 scihub 實作；當 paper-search-mcp 的 `download_scihub` 因 mirror 故障失敗時的二備 |

### 2.4 付費牆論文

**使用者手動上傳 PDF** 到 `原始PDF/`，流程同下。

### 2.5 PDF → Markdown 轉檔

拿到 PDF 後用 **`mcp__llamaparse__parse_pdf_to_markdown`**（agentic tier）轉成 Markdown，存到 `原始PDF/*.md`。此時本地有「可程式全文檢索」的底本。

**驗收（關鍵）**：每個下載的 PDF 必須打開檢查內容語言 / 標題 / 作者是否與 metadata 一致——**防 sci-hub 映射錯誤（Polluted PDF）導致的資料污染型幻覺**。paper-search-mcp 的 OA-first chain 把 sci-hub 放在最後、降低發生率，但一旦觸發還是要肉眼驗。

---

## ★ 第二輪深挖（核心步驟）

> **第一輪只是入門票。第一批 PDF 內文的 References 才是真正的核心文獻來源。**

讀完第一輪 PDF 後執行：

### Step 1：從本地 MD 抽 references
- 用 Grep 搜尋每篇 MD 的 References / Bibliography 段落
- 對每篇 PDF 內容做摘要時，請 LLM **明確列出該文最關鍵的 5–10 個 cited references**（含 DOI / PMID / 作者年份）
- 特別關注：
  - 被多篇 review 共同引用的 seminal paper
  - 第一輪論文用來支撐核心論斷的 primary study
  - 內文出現 "landmark trial"、"first to show"、"largest cohort" 等字眼附近的引用

### Step 2：把這些 references 餵回第一層
- 對每筆候選 reference 優先用 `mcp__paper-search__search_papers`（多源並行）驗證真實性
- 有 DOI 時 `search_crossref` 直查最快
- 若是被高頻共引（≥2 篇第一輪論文都引到），直接列為「必拿全文」

### Step 3：跑第二層全文下載
- 對「必拿全文」清單直接跑 `mcp__paper-search__download_with_fallback`
- LlamaParse 轉 MD，加入本地語料庫

### Step 4：可選的第三輪
- 若第二輪論文又出現新的 seminal references 且該文本身極關鍵 → 再做一輪
- 多數 topic 兩輪就足夠（第三輪以後通常邊際效益遞減）

**輸出**：本地 `原始PDF/` 內含第一輪 + 第二輪（必要時第三輪）的全文 MD。

---

## 第三層｜幻覺校正（逐一比對，寧缺勿濫）

寫作階段 LLM 只能引用第二層已落地的本地 MD。交稿後做三路稽核：

1. **DOI 存活檢查**：每個 DOI 用 `https://doi.org/{doi}` resolve（HTTP HEAD 或瀏覽器），404 標記移除；或用 `search_crossref(query="<DOI>")` 確認存在
2. **內容匹配檢查**：論文 metadata（title / 作者 / 期刊 / 年份）必須與 `search_pubmed` / `search_crossref` 回傳結果一致
3. **數字 / 論斷 Grep**：文中所有百分比、樣本數、OR/HR 值、引號內字串，必須能在對應的本地 MD 中 Grep 到——Grep 不到就降級為「待確認」或移除

---

## 幻覺類型清單（稽核時逐項檢查）

| # | 類型 | 範例 | 線索 |
|---|---|---|---|
| 1 | Fabricated citation | 論文根本不存在 | DOI 404、`search_pubmed` / `search_crossref` 查無 |
| 2 | DOI mismatch | DOI 可解析但指向別篇 | DOI 的 title/journal ≠ 文中所寫 |
| 3 | Author attribution error | 對的論文、錯的作者 | Crossref / PubMed metadata 比對 |
| 4 | Journal/year shift | 期刊或年份誤植 | metadata 比對 |
| 5 | Fabricated statistic | 「GADA 陽性 87%」原文寫 78% | 本地 MD 全文 Grep 不到 |
| 6 | Claim misattribution | 引 [3] 但論點實際出自 [7] | 原文未出現該論斷 |
| 7 | Synthesis overreach | 「A 證明 X，故 Y」—A 只證到 X 前提 | 邏輯鏈審查 |
| 8 | Quote fabrication | 把改寫硬放引號 | 引號內字串 Grep 不到 |
| 9 | Temporal anachronism | 引 2026 年寫 2024 年事件 | 時序不合 |
| 10 | Confabulated detail | RCT 變 cohort、n 從 133 變 1330 | 對照原文 Methods |
| 11 | Ghost consensus | 「多數研究顯示…」但只有 1–2 篇 | 要求回溯來源 |
| 12 | Dead link to self | 引 [25] 但參考列表只到 [24] | range check |
| 13 | **Polluted PDF** | sci-hub 映射錯誤回傳完全不相干的 PDF | 開檔檢查內容語言 / 標題 |

---

## 真實案例（前車之鑑）

### 案例 1：DOI 期刊寫錯（T1DM vs LADA, 2026-04）
LLM 初稿引用 Noble 2024 為 *Diabetologia* DOI——404。實為 *Frontiers in Immunology*。**期刊完全寫錯**，靠 DOI 存活檢查攔下。

### 案例 2：sci-hub 映射錯誤（資料污染）
sci-hub 根據 DOI `10.1016/j.dsx.2021.102197` 回傳的 PDF 內容竟是 2005 年葡萄牙文哲學論文。**檔名對、內容錯**。若不打開驗證，LlamaParse 會把它轉成「看似正常」的 MD，再被 LLM 當成研究結論引用——典型的「資料污染型幻覺」。paper-search-mcp 把 sci-hub 放 fallback chain 最末雖降低觸發率，一旦觸發仍需驗證。

### 案例 3：PubMed MCP session 斷線（2026-04）
舊 `claude.ai PubMed` connector 偶發 `MCP session has been terminated`，造成稽核中斷。改用 `paper-search-mcp`（stdio 本機執行）後不再依賴 Anthropic 端的遠端 session，可用性顯著提升。

---

## 標準輸出格式（Notion / Markdown 共用）

每個引用後加符號：
- 📄 = **本地全文驗證**（第二層 / 第二輪都拿到 PDF + Grep 通過）
- 📌 = **僅 abstract / metadata**（付費牆未過，禁止對其內容做任何具體斷言）

每個 topic 資料夾結構建議：
```
{topic}/
├── {topic}.md                  # 主文
├── 撰寫方法論.md                # 本流程的當次套用紀錄
├── MISSING_FULLTEXT.md         # 標記未拿到全文的 DOI 與原因
└── 原始PDF/
    ├── {author}_{year}.pdf
    └── {author}_{year}.md      # LlamaParse 轉檔
```

---

## 可重用資產

- **paper-search-mcp**（<https://github.com/openags/paper-search-mcp>）：本 skill 的主引擎；21 來源並行搜尋 + OA-first 下載鏈
- **scihub-mcp**（<https://github.com/zinojeng/sci-hub>）：Playwright headless Chromium 繞 DDoS-Guard；作為 paper-search-mcp `download_scihub` 的二備
- **llamaparse-mcp/auto_convert.py**：掃 `WATCH_DIRS` 中尚未轉檔的 PDF，先 call paper-search-mcp / scihub 補缺，再 LlamaParse 轉 MD
- **MISSING_FULLTEXT.md 協定**：每個專案 `原始PDF/` 旁放一份，auto_convert.py 自動讀取表格中的 DOI

### paper-search-mcp 必要 / 建議環境變數

放在 `~/paper-search-mcp/.env` 或專案 `.env`：

```dotenv
# 必要
PAPER_SEARCH_MCP_UNPAYWALL_EMAIL=your@email.com

# 建議（顯著提升穩定性 / 額度）
PAPER_SEARCH_MCP_CORE_API_KEY=<free_from_core.ac.uk>
PAPER_SEARCH_MCP_SEMANTIC_SCHOLAR_API_KEY=<free_from_semanticscholar.org>

# 選用
PAPER_SEARCH_MCP_GOOGLE_SCHOLAR_PROXY_URL=<http_proxy_url>
PAPER_SEARCH_MCP_DOAJ_API_KEY=
PAPER_SEARCH_MCP_ZENODO_ACCESS_TOKEN=
```

---

## Agent Team 擴展（v2.1.32+，嚴格稽核時啟用）

> 文獻回顧本質上是「研究 + 並行敵對審查」——這正是 agent team 的核心優勢場景（見官方 docs/agent-teams：research and review、competing hypotheses）。

### 啟用前置

在 `~/.claude/settings.json` 加入：
```json
{ "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" } }
```
需 Claude Code v2.1.32+。建議 macOS 用 `tmux -CC` 進 iTerm2 取得 split panes。

### 為什麼用 agent team 而不是 subagent？
- **Subagent**：單向 spawn → 回報，無法互相辯論
- **Agent team**：每個 teammate 獨立 context，**互相發訊息 + 共享 task list**，可跑「敵對稽核」（adversarial peer review）——這正是攔下幻覺需要的結構

### 兩階段團隊設計

#### Phase A｜Writing Pipeline（多半串行，少量並行）

| 角色 | 任務 | 模型建議 |
|---|---|---|
| **outline-architect** | 讀使用者指令 + 已有 PDF 摘要，產出章節 outline 與每章「必須回答的問題」 | sonnet |
| **literature-scout** | 跑第一層：`mcp__paper-search__search_papers` 多源並行 + `openevidence` + `consensus`，產出引用清單 + 為每筆確認 PMID/DOI | sonnet |
| **fulltext-fetcher** | 跑第二層：`mcp__paper-search__download_with_fallback` 為主、`scihub-mcp` 為二備 + LlamaParse 轉 MD；**並打開 PDF 肉眼驗證內容**（防 Polluted PDF） | sonnet |
| **deep-dive-scout** | ★ 第二輪：從第一輪本地 MD 抽 references → 高頻共引者重跑 scout + fetcher | sonnet |
| **drafter** | 只引用本地 MD，寫初稿；每個論斷後加 `[本地MD檔名]` 註記供後續 Grep | opus |
| **counter-advocate** | 對 drafter 的每個論斷找反例文獻；存在反證者標 ⚠️ 並要求 drafter 改寫 | opus |
| **style-editor** | 統一語氣、收斂冗詞、確認 📄/📌 標記 | sonnet |

Lead 的 spawn 順序：架構師先單獨跑 → scout + fetcher 並行 → drafter 等兩者完成 → counter-advocate 與 drafter 通訊辯論 → style-editor 收尾。

#### Phase B｜Audit Pipeline（**完全並行的敵對稽核**）

> 仿 docs 的 "competing hypotheses" 模式：三個稽核員分別從不同維度攻擊草稿，arbitrator 收斂結論。

| 角色 | 任務 | 通過標準 |
|---|---|---|
| **citation-verifier** | 對每個 DOI 做 `https://doi.org/{doi}` resolve；用 `mcp__paper-search__search_crossref` / `search_pubmed` 比對 metadata | DOI 200 + title/作者/期刊/年份全對 |
| **claim-auditor** | 文中每個百分比、樣本數、OR/HR、引號字串，到對應本地 MD 跑 Grep | Grep 命中率 100%；未命中者列出供 drafter 處理 |
| **cross-referencer** | 對「[5] 顯示…」這類句子，驗證參考列表 [5] 真的支撐該論斷；檢查序號 range、ghost consensus、claim misattribution | 13 類幻覺逐項過 |
| **arbitrator** | 收三人報告，分級為「必修 / 待議 / 通過」；對「必修」逐條呼叫 drafter 修改並回跑稽核 | 全部歸零 |

三個稽核員 spawn 時用 broadcast 一次發任務，獨立跑、互相挑戰彼此漏看的點，arbitrator 收斂後 lead 才能交稿。

### Lead 的 spawn 樣板（可直接複製）

**啟動 Phase A**：
```
建立 agent team 撰寫「{topic}」文獻回顧。共 7 個 teammate：
outline-architect、literature-scout、fulltext-fetcher、deep-dive-scout、drafter、counter-advocate、style-editor
（皆使用對應的 subagent 定義）。流程依 SKILL.md Phase A 順序。
所有 teammate 都必須遵守 medical-literature-review skill 的硬規則：
LLM 只做改寫，事實必須能 grep 驗回 原始PDF/*.md。
搜尋引擎：paper-search-mcp（search_papers / search_pubmed / search_crossref 等）；下載引擎：download_with_fallback。
工作目錄：{topic 資料夾絕對路徑}
```

**啟動 Phase B（草稿完成後）**：
```
草稿在 {topic}.md。spawn 4 個 teammate 做敵對稽核：
citation-verifier、claim-auditor、cross-referencer 並行；arbitrator 收斂。
citation-verifier 用 mcp__paper-search__search_crossref / search_pubmed 驗 metadata。
require plan approval：arbitrator 在分級「必修」時必須先提交修正計畫給我審。
全部 issue 歸零前不得 cleanup team。
```

### Hooks 強化（可選）

設 `TaskCompleted` hook：當 drafter 把任務標 completed 前，自動跑 grep 腳本檢查所有 `[本地MD檔名]` 註記是否確實存在；不存在則 exit 2 擋下並回傳訊息。

### 何時不要用 agent team
- 單篇 case report、小範圍更新（token 成本不划算）
- 引用 < 10 篇的短文（單 session + 第三層手動稽核就夠）
- 機構付費牆檔案佔多數時（fetcher 大半會卡，不如先手動上傳完整 PDF 再啟動）

### 對應的 subagent 定義檔位置
本 skill 同時建立了 11 個可重用 subagent 定義在 `~/.claude/agents/`：
`outline-architect.md`、`literature-scout.md`、`fulltext-fetcher.md`、`deep-dive-scout.md`、`drafter.md`、`counter-advocate.md`、`citation-verifier.md`、`claim-auditor.md`、`cross-referencer.md`、`arbitrator.md`、`style-editor.md`。
Lead 用名稱引用即可，不必每次重寫 system prompt。
**升級注意**：v2.0 後應檢查這些 agent 定義中提到的 `mcp__claude_ai_PubMed__*` 工具名是否已全部替換為 `mcp__paper-search__*`。

---

## 三向同步：本地 ↔ Notion ↔ Google Drive

完成主文後，產物要同步到三個目的端，**本地為 source of truth**：

### 目錄結構（鏡像式）
```
本地    Documents/Notion/醫學/{科別}/{topic 父}/{topic}/
Notion  {對應父 page} → 新建 child page「{topic}」
Drive   ~/Library/CloudStorage/GoogleDrive-*/我的雲端硬碟/醫學/{科別}/{topic 父}/{topic}/
```

三端的子結構固定為：
- `{topic}.md` — 主文
- `_citations_round{N}.md` — 各輪引用清單
- `MISSING_FULLTEXT.md` — Polluted PDF / paywall 紀錄
- `原始PDF/` — PDF + LlamaParse / pdftotext 轉的 MD
- `SYNC_LINKS.md` — 三端路徑索引

### 同步步驟
1. **Notion**：
   - `notion-search` 找父 page（例：`Hypothyroidism`、`T1DM`）
   - `notion-create-pages` 以 `parent.page_id` 建 child；content 用主文 markdown
   - 主文超過 API 單次限制 → 先建頁再用 `notion-update-page` 分段 append
2. **Google Drive**（macOS Drive for Desktop 已掛載時）：
   - `mkdir -p` 目標資料夾
   - `cp` 主文 + `_citations_*.md` + `MISSING_FULLTEXT.md` + 整個 `原始PDF/`
   - Drive 客戶端會自動上傳；不必另外呼叫 API
3. **建立 `SYNC_LINKS.md`**：本地與 Drive 各放一份，內含三端路徑與 Notion page URL

### 何時跳過某一端
- Notion 無對應父頁 → 先問使用者要建在哪
- Drive 未掛載 / 無對應帳號 → 改用 `rclone` 或寫 `SYNC_LINKS.md` 等使用者手動上傳

### 文末 footer 慣例
主文最後一行加：
```
*同步位置：本地 `{path}` ↔ Notion `{title}` ↔ Google Drive `{path}`*
```

---

## LlamaParse credit 用盡的 fallback

LlamaParse 是首選（agentic 解析品質佳），但有 credit 上限。若 batch 回 `Error 402: exceeded maximum number of credits`：
- 優先改用 paper-search-mcp 的來源內建 `read_*_paper` 工具（arXiv / PMC / Europe PMC / bioRxiv / medRxiv 都有 OA 全文 read）
- 或用 `pdftotext -layout {file}.pdf {file}.md`（poppler，macOS `brew install poppler`）
- 品質次於 LlamaParse 但對 reference list / 表格仍可用；圖片 caption / 多欄複雜版型會劣化
- **不論用哪個轉換器，都要肉眼驗證 PDF 第一頁標題與 metadata 一致** — 防 Polluted PDF 不分轉換工具

---

## 套用 checklist（每個 topic 開工前過一次）

- [ ] 第一層：**paper-search-mcp `search_papers` 多源並行**（預設） + OpenEvidence + Consensus + 必要的 `search_pubmed` / `search_crossref` 補網，產出引用清單
- [ ] 第二層：清單上的論文用 **`download_with_fallback`** 跑 OA-first chain，失敗再試 `scihub-mcp` / 使用者上傳 PDF；全 PDF 經 LlamaParse 轉 MD
- [ ] 開檔肉眼驗證：每個 PDF 的標題 / 語言 / 作者與 metadata 一致（防 Polluted PDF）
- [ ] **第二輪深挖**：每篇本地 MD 的 References 抽出，高頻共引者重跑第一層 + 第二層
- [ ] 寫作：LLM 只引用本地 MD 內的論文與數字；案例的**人口學資訊（年齡、性別、BMI）必須 grep 對齊原文**，不能想當然
- [ ] 第三層稽核：DOI resolve + `search_crossref` / `search_pubmed` metadata 比對 + 數字 Grep + 13 類幻覺逐項過
- [ ] 標記 📄 / 📌；未過全文者寫進 `MISSING_FULLTEXT.md`
- [ ] **三向同步**：本地 → Notion → Drive；建 `SYNC_LINKS.md`
- [ ] 在 topic 資料夾留一份 `撰寫方法論.md` 紀錄當次套用結果（成品 % / 攔下的幻覺案例）

---

## v1.x → v2.0 工具名對照（升級時速查）

| 舊（v1.x, `claude.ai PubMed` connector） | 新（v2.0, `paper-search-mcp`） |
|---|---|
| `mcp__claude_ai_PubMed__search_articles` | `mcp__paper-search__search_pubmed` 或 `mcp__paper-search__search_papers` |
| `mcp__claude_ai_PubMed__get_article_metadata` | `mcp__paper-search__search_pubmed(query="<PMID>")` 或 `search_crossref` |
| `mcp__claude_ai_PubMed__lookup_article_by_citation` | `mcp__paper-search__search_papers(query="<title> <author> <year>")` |
| `mcp__claude_ai_PubMed__find_related_articles` | `mcp__paper-search__search_semantic`（引用圖譜）+ `search_openalex` |
| `mcp__claude_ai_PubMed__get_full_text_article` | `mcp__paper-search__download_with_fallback` → LlamaParse |
| `mcp__claude_ai_PubMed__convert_article_ids` | `mcp__paper-search__search_crossref(query="<DOI or PMID>")`（回傳 metadata 含交叉 ID）|
| `mcp__research_hub__download_paper` | `mcp__paper-search__download_with_fallback`（OA-first chain 已內建）|
| `mcp__research_hub__download_papers_batch` | `download_with_fallback` 逐篇呼叫；或 paper-search CLI `paper-search download` 批次 |

---
> Source: [zinojeng/medical-literature-review](https://github.com/zinojeng/medical-literature-review) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
