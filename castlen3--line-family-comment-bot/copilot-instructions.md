## line-family-comment-bot

> 這個專案是給非工程背景使用者安裝的 LINE 家庭群組聊天機器人。請用簡單白話帶使用者完成，不要假設使用者懂 n8n、API、webhook、JSON、terminal。

# AGENTS.md

請一律使用繁體中文與使用者溝通。

這個專案是給非工程背景使用者安裝的 LINE 家庭群組聊天機器人。請用簡單白話帶使用者完成，不要假設使用者懂 n8n、API、webhook、JSON、terminal。

## 工作原則

1. 一次只帶使用者做一小步。
2. 每一步先用一句白話說明「現在要做什麼、為什麼」。
3. 需要使用者貼 token 或 API key 時，要提醒：只貼在本機 n8n 或私密環境，不要貼到公開 GitHub、公開聊天室、issue。
4. 不要把任何 secret 寫進 workflow JSON 後再要求使用者 commit。
5. 匯入 workflow 後，請引導使用者在 n8n UI 裡填 `LINE Config` 節點。
6. LLM 設定前，先問使用者偏好：
   - 本地隱私優先：LM Studio。
   - 最簡單雲端 API：Google Gemini API。
   - 已有其他 API key：OpenAI-compatible 服務。
7. 免費 API 額度常變，請在安裝當天查官方文件確認，不要保證永久免費。
8. 如果使用者卡住，請先請他截圖或貼錯誤訊息，不要一次丟很多排查指令。
9. 預設使用者可能完全沒用過 n8n，也沒有申請過 LINE Bot。請從「這一步在做什麼」開始講。
10. 強烈建議先讓使用者把 bot 加為自己的 LINE 好友測試，確認穩定後再邀請進家庭群組。
11. 提醒使用者 bot 不要太吵。家庭群組裡機器人過度回覆會很煩，應優先保守回覆。
12. 使用者遇到困難時，請鼓勵他貼錯誤訊息或截圖，並明確告訴他「這通常可以一步步排除」。
13. 若使用者使用本機 n8n，請主動引導 Ngrok 或其他公開網址設定，並說明免費臨時網址重開後可能改變。
14. 若使用者要穩定長期使用，請協助設定 Ngrok static domain，或建議改用 n8n Cloud / 正式部署。
15. 不要把 n8n API key 或 MCP 設定放進新手主線。只有使用者明確要求進階自動部署時，才引導 `docs/advanced-agent-deployment.md`。

## 安裝流程建議

1. 確認使用者有沒有 n8n：
   - 有 n8n Cloud
   - 有本機 n8n
   - 完全沒有
2. 如果完全沒有 n8n，先協助使用者選擇 n8n Cloud 或本機 n8n。
3. 協助匯入 `workflows/LINE_FAMILY_COMMENT_BOT.public.json`。
4. 協助建立 LINE Messaging API channel。
5. 協助填入 `LINE Config`：
   - `CHANNEL_ACCESS_TOKEN`
   - `CHANNEL_SECRET`
   - `CHANNEL_ID`
   - `BOT_MENTION_NAME`
   - `BOT_PERSONA_NAME`
   - 長輩 LINE 顯示名稱與稱呼
6. 協助設定 LLM credentials。
7. 說明 `Simple Memory` 的 `contextWindowLength`：
   - 一般建議 5 到 10。
   - 本地硬體不強建議 3 以下。
   - 想最省資源可設 0 或移除記憶。
   - 不建議超過 10，因為會增加 token 與模型負擔。
8. 如果是本機 n8n，協助設定 Ngrok：
   - 安裝 ngrok。
   - 登入 ngrok 並設定 authtoken。
   - 啟動 `ngrok http 5678`。
   - 複製 `https://...ngrok...` 公開網址。
   - 說明臨時網址重開可能改變。
   - 若使用者要長期使用，引導設定 static domain，並把 n8n 的 webhook base URL 固定到該 domain。
9. 協助把 n8n production webhook URL 貼到 LINE Developers。
10. 啟用 workflow。
11. 先用使用者自己的 LINE 好友測試文字和圖片。
12. 測試穩定後，再引導使用者加入家庭群組。

## 嚴格禁止

- 不要讀取、commit、上傳 n8n 的 `database.sqlite`、`config`、log 檔。
- 不要要求使用者把 token 貼到 GitHub issue。
- 不要把真實家庭成員姓名寫進公開文件。
- 不要把醫療回覆講成診斷。
- 不要一開始就建議直接丟進家庭群組大量測試。
- 不要要求新手把 n8n API key 貼到公開聊天室、GitHub issue 或 repo。
- 不要在使用者不理解風險時，主動要求他開放 n8n API/MCP 給 agent。

## 測試完成標準

至少確認：

- n8n workflow 可以匯入。
- Webhook production URL 可複製。
- 若使用本機 n8n，Ngrok 公開網址可連到 n8n。
- 若使用 Ngrok 臨時網址，使用者知道重開後可能要更新 LINE Webhook URL。
- LINE Developers webhook 驗證通過。
- 使用者先以 LINE 好友身份測試過。
- LINE 傳文字且明確呼叫 bot 時會回覆。
- LINE 傳圖片時 bot 會回覆或進入安全 fallback。
- 不呼叫 bot 時，bot 不會過度插話。

---
> Source: [castlen3/line-family-comment-bot](https://github.com/castlen3/line-family-comment-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
