## chrome-driver

> 我想製作一套可供多數人使用的 Chrome 自動化工具，請用 Python 撰寫，工具的需求如下：

我想製作一套可供多數人使用的 Chrome 自動化工具，請用 Python 撰寫，工具的需求如下：


1. 程式會自動尋找與主程式同目錄的 chromedriver.exe，不需要手動輸入路徑或安裝環境變數。整體設計需能打包為 EXE 後直接運作。

2. 所有操作步驟來自同目錄下的 command.txt，每行一個指令，格式為：
   指令名 = 參數
   指令與參數之間用等號（=）分隔。若參數有多個欄位，用兩個直線符號（||）區隔。

   支援的指令如下：
OPEN_URL = [網址]
LOGIN = [帳號]||[密碼]
CLICK_BY_TEXT = [畫面文字]
WAIT = [秒數]
WAIT_FOR_TEXT = [要等待出現的文字]
IF_TEXT [條件文字] THEN CLICK_BY_TEXT [畫面文字]

範例:
開啟chrome
 輸入 google.com
輸入張惠妹
搜尋
下載一張照片



3. 程式需具備基本的錯誤處理與 LOG 紀錄功能：
   每個步驟的執行結果應即時顯示在畫面上，並同步寫入 log.txt，格式如下：
   [時間] 指令內容 = PASS
   [時間] 指令內容 = FAIL（附上錯誤原因）

4. 執行過程需模擬人類行為以避開自動機器人偵測，包括：

   * 使用非 headless 模式啟動 Chrome
   * 模擬滑鼠移動或點擊
   * 執行前後加上隨機 sleep（例如 1\~3 秒）
   * 可選擇載入使用者的 Chrome Profile（使用 user-data-dir）

5. 介面設計需簡潔清楚，使用 tkinter 製作簡單 GUI：

   * 提供按鈕可執行指令檔
   * 顯示目前執行的指令內容
   * 顯示即時的執行結果 log
   * 整體流程無需終端機操作，讓非技術使用者也能直接使用

請將指令解析與執行邏輯設計為可擴充的架構，未來可支援更多指令類型與參數控制。程式名稱建議為 chrome_automation_tool.py

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JOVIANpega) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
