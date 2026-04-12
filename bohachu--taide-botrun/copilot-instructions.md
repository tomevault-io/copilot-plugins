## taide-botrun

> - 參考時機：使用 WebFetch 抓取 PDF 檔案時、看到 "maxContentLength size exceeded" 錯誤時、規劃批次抓取教育資源時、建立資料收集流程時

# TAIDE-Botrun 專案 Claude 指令

## 地雷經驗記憶

### WebFetch 大型 PDF 檔案限制
- 參考時機：使用 WebFetch 抓取 PDF 檔案時、看到 "maxContentLength size exceeded" 錯誤時、規劃批次抓取教育資源時、建立資料收集流程時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-WebFetch大型PDF檔案限制.txt
- 核心要點：
  - WebFetch 工具有 10 MB 檔案大小限制
  - 大型 PDF（如學測解析）需使用 curl/wget 下載後再用 Read 工具處理
  - 建立錯誤處理機制，WebFetch 失敗時自動切換下載方案

### WebFetch SSL 憑證與大小計算問題
- 參考時機：使用 WebFetch 抓取網頁時、看到 "unable to verify the first certificate" 或 "sizeCalculation return invalid" 錯誤時、批次抓取教育網站資料時、網頁包含圖片重要資訊時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-WebFetch-SSL-憑證與大小計算問題.txt
- 核心要點：
  - 某些網站 SSL 憑證配置不完整會導致抓取失敗
  - 網頁內容過大或結構複雜可能觸發大小計算錯誤
  - 重要內容若以圖片呈現無法直接提取，需查閱官方文件或其他來源
  - 建議平行抓取多個來源，標記狀態（success/failed/partial），交叉驗證資料

### WebFetch 網站防爬蟲限制（403 Forbidden）
- 參考時機：使用 WebFetch 抓取網頁時、看到 403 Forbidden 錯誤時、抓取 Medium 等付費平台內容時、規劃批次網頁抓取策略時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-WebFetch網站防爬蟲限制.txt
- 核心要點：
  - Medium 等網站有防爬蟲機制會回傳 403 Forbidden，WebFetch 無法處理
  - 優先使用 MCP web fetch 工具（較少限制），次選 curl/wget 加 User-Agent，最後手動處理
  - 規劃階段需評估網站爬蟲友善度，準備多種資料收集方法
  - 本次任務成功率 77.8%（7/9），需建立「待手動處理」清單處理特殊案例

### 網頁資料抓取限制（反爬蟲、版權保護、工具限制）
- 參考時機：使用 WebFetch 抓取 Wikipedia 時、抓取電子書平台內容時、看到 403 錯誤時、需要大量結構化資料時、規劃知識體系資料收集時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-網頁資料抓取限制.txt
- 核心要點：
  - Wikipedia 因反爬蟲機制會回傳 403 錯誤，應使用官方 API（action=query&prop=extracts）
  - 電子書平台因版權保護僅能抓取平台資訊，需遵守使用限制
  - WebFetch 設定具體 prompt 可改善提取效果（明確要求欄位、結構化呈現）
  - 無法抓取時仍記錄失敗狀態於 JSONL，方便後續手動處理
  - 本次知識體系抓取成功率 91.7%（11/12），Wikipedia 失敗但其他台灣官方資源順利完成

### 答案解析資料抓取圖片內容處理
- 參考時機：抓取教育資源（試題、解析、課綱）時、目標網站以圖片發布版權內容時、需要結構化資料但來源為圖片時、規劃大規模資料收集流程時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-答案解析資料抓取圖片內容處理.txt
- 核心要點：
  - 多數教育網站（大學問、聯合新聞網）將詳細解析以圖片形式呈現，WebFetch無法直接提取文字
  - 標記為 status: "partial"，在 notes 記錄圖片數量與格式
  - 優先抓取官方PDF檔案（如大考中心）或提供文字版解析的網站
  - 明確區分success（文字）、partial（圖片）、failed（無法抓取）
  - 本次9個資源100%完成，但僅33%提供結構化文字（3/9）

### 目錄路徑與中文編碼問題
- 參考時機：在專案中建立新目錄前、使用 curl/wget 下載檔案到中文路徑時、遇到 "No such file or directory" 且路徑顯示亂碼時、批次處理檔案前需要確認目錄結構時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-目錄路徑與中文編碼問題.txt
- 核心要點：
  - 專案中可能存在名稱相似的多個目錄（如 step-002-抓取原始資料 vs step-002-資料抓取），任務前需用 ls/find 確認
  - curl 無法處理路徑中的中文字元，macOS 終端機會顯示編碼錯誤，需使用英文檔名
  - 發現目錄錯誤後用 mv 移動檔案，用 git status 確認檔案位置正確
  - 檔案命名優先使用英文、數字、連字號、底線，避免空格和特殊字元

### Git 遠端倉庫未建立
- 參考時機：執行 git push 時出現 "Repository not found" 錯誤、新專案初次推送時、git remote -v 顯示 URL 但 push 失敗時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-Git遠端倉庫未建立.txt
- 核心要點：
  - 本地 Git 已初始化並有 commits，但 GitHub 上尚未建立對應倉庫
  - 遠端 URL 已設定，但倉庫實際不存在會導致 push 失敗
  - 解決方案：(1) 在 GitHub 上手動建立倉庫；(2) 使用 `gh repo create` 自動建立；(3) 先保留本地 commit 等待手動建立
  - 預防措施：專案開始前先建立 GitHub 倉庫，或使用 `gh repo create` 自動建立並連結

### 題目生成多正確答案問題
- 參考時機：生成字音辨識題目時、設計多音字題目時、題目審查階段、看到 metadata.notes 包含「需重新設計」時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-題目生成多正確答案問題.txt
- 核心要點：
  - 題目設計前必須先用 WebSearch 驗證所有選項的讀音,確認教育部標準讀音
  - 確保每題只有唯一正確答案,避免多個選項都符合題幹要求
  - 常見問題：「何者相同」題型所有選項都不同、「何者正確」題型多個選項都正確
  - 解決流程：先驗證讀音 → 設計選項 → 反向檢查 → 記錄驗證來源
  - 本次生成30題發現約27%存在邏輯問題,需要建立題目審查機制

### 修辭手法題目生成經驗
- 參考時機：生成修辭格題目時、設計需要精確概念區分的題目時、需要設計有教學價值的陷阱選項時、建立系統性題目生成流程時、生成複合修辭題時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-修辭手法題目生成經驗.txt
- 核心要點：
  - 系統性涵蓋所有必要修辭格（10種主要修辭格，每種2-4題）
  - 精確區分易混修辭格（明喻vs暗喻vs借喻、排比vs對偶vs層遞、反襯vs對比、諧音vs語義雙關、提問vs激問vs懸問、擬人vs自然描寫）
  - 陷阱設計來自真實學習困難點，每個陷阱都包含 trap_type、why_wrong、how_to_avoid、common_mistake_rate
  - 選擇經典例句（唐詩宋詞、學測常考篇章），避免爭議性例句
  - 複合修辭要完整分析，避免過度解讀
  - 本次生成30題（RH-001到RH-030），涵蓋10種修辭格，難度分布：Level 3（50%）、Level 4（43%）、Level 5（7%）

### Bash 腳本與 zsh 相容性問題
- 參考時機：撰寫 shell 腳本時、在 macOS 上執行 .sh 腳本時、看到 `declare: -A: invalid option` 或 `parse error near '('` 錯誤時、腳本執行結果與預期不符時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-Bash腳本與zsh相容性問題.txt
- 核心要點：
  - macOS 預設 shell 是 zsh，bash 特有語法（如 `declare -A` 關聯陣列）不支援
  - 對於檔案處理任務，優先使用 Node.js (.mjs) 替代 shell 腳本
  - 如必須用 shell，保持腳本簡單，使用 `#!/bin/bash` 並以 `bash script.sh` 執行
  - 建立診斷腳本先確認資料狀態，再執行合併等操作

### OpenRouter API LLM 模型測試經驗
- 參考時機：使用 OpenRouter API 測試 LLM 時、測試模型對台灣繁體中文的理解能力時、比較不同模型在特定領域表現時、設計 LLM 測試腳本時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-OpenRouter-API-LLM測試經驗.txt
- 核心要點：
  - 免費模型（如 Gemma 3 12B IT :free）可能不回報完整 token 使用資訊（output_tokens=0）
  - 文學史知識兩模型都有錯誤（建安七子題），顯示中文文學訓練不足
  - Gemini 字形辨識較佳（2/2），Gemma 繁體字形訓練不足（0/2）
  - 回應速度差異大：Gemini 0.76s vs Gemma 3.89s
  - 自動生成題目必須先用網路資料驗算，假設原答案可能錯誤

### 上下文工程 DSL 設計經驗
- 參考時機：設計知識驅動的 LLM 解題系統時、建立知識庫與模板架構時、使用 ripgrep 做平行檢索時、LLM 解題正確率不如預期時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-上下文工程DSL設計經驗.txt
- 核心要點：
  - 知識庫標記「變體」會被 LLM 誤解為「也算對」，應改用「錯」明確標記
  - LLM 會自行推測知識庫未涵蓋的細節並可能推測錯誤，需在模板中強調「只根據知識庫判斷」
  - 字形題 LLM 容易誤讀「」內文字，模板需加入格式說明範例
  - 平行 rg 檢索時需建立常見形近字組列表，匹配任一字時加入整組關鍵字搜尋
  - 成果：Gemma 3 12B IT 達成 5/5 正確 (100%)，平行 rg 檢索速度 10-25ms/題

### LLM 關鍵字提取與知識檢索系統設計
- 參考時機：設計 RAG 架構時、使用 LLM 提取關鍵字時、rg 檢索結果為空但手動測試有結果時、LLM 不理解題目格式時
- 文件位置：/Users/40gpu/coding_projects/taide-botrun/docs/地雷-LLM關鍵字提取與知識檢索系統設計.txt
- 核心要點：
  - 硬編碼關鍵字列表（includes() + 對照表）無法泛化，應改用 LLM 多角度提取
  - rg 檢索 `keyword.length < 2` 會跳過單一中文字（如「厲」），需加入 hasChinese 判斷
  - 成語題「錯誤／正確」格式需在 template 明確說明，否則 LLM 只分析後者
  - 多選項都正確時需在知識庫加入「選擇題判斷要點」指導 LLM 選最佳答案
  - v3 架構：LLM 提取關鍵字 + 平行 rg 檢索 + LLM 解題，達成 100% 正確率

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bohachu)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bohachu)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
