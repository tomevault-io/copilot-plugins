## trip-planner

> | Model | `gemini-2.5-flash` |

# Gemini API Integration Specs (AI 整合規範)

## 模型與端點 (Model & Endpoint)

| 項目 | 值 |
|------|-----|
| Model | `gemini-2.5-flash` |
| Endpoint | `POST https://generativelanguage.googleapis.com/v1beta/models/{MODEL}:generateContent?key={API_KEY}` |

## 呼叫配置 (Generation Config)

- **Temperature:** `0.1`（極低溫度，確保 JSON 結構穩定，減少幻覺）
- **responseMimeType:** 不可設定（與 `google_search` tool 不相容，API 會回傳 400 錯誤）
- **Tools:** 必須啟用 `[{ google_search: {} }]` 以獲取真實世界資訊。

## 提示詞工程 (Prompt Engineering Protocol)

為了確保輸出可被前端系統消化，Prompt 必須包含以下強制性指令：

- **嚴格 JSON 約束：** 「只允許輸出一個 JSON 物件，嚴禁包含任何 Markdown 標記（如 ` ```json `）或其他文字/引言！」
- **經緯度約束：** 「請務必在 `lat` 與 `lng` 欄位給予該景點真實的 GPS 經緯度座標（浮點數），以便在地圖上渲染。」
- **枚舉值約束：** 明確列出 `type`（e.g., `sightseeing`, `dining`）與 `category`（e.g., `info`, `warning`）允許的字串。

## 暴力 JSON 擷取引擎 (Bulletproof Parsing)

AI 的回覆常帶有不可控的文字。代理人必須使用以下邏輯解析 API 回應：

```js
let text = responseText;
const firstBrace = text.indexOf('{');
const lastBrace = text.lastIndexOf('}');
if (firstBrace !== -1 && lastBrace !== -1) {
  text = text.substring(firstBrace, lastBrace + 1);
}
const parsedData = JSON.parse(text);
```

## 退避重試機制 (Exponential Backoff)

- API 呼叫必須包裝在 `try...catch` 與 `for` 迴圈中。
- **最大重試次數：** 3 次。
- **延遲策略：** `await delay(1500 * (i + 1))`（1.5s, 3s）。若 3 次皆失敗，拋出錯誤並觸發 Fallback 演算法。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/neobradley)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/neobradley)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
