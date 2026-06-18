## mantra-api-frontend

> Copilot AI 協作指引 - 程式碼風格與開發原則


# Copilot 指引

## 核心原則

**簡單至上**：不要過度設計功能和架構，保持程式碼簡潔易懂

## 程式碼風格

### 一般原則

- 程式只做好一件事
- 避免不必要的註解（程式碼應該自我說明）
- 使用小寫字母並儘量簡短
- 資料應該儲存為文字檔案
- 可移植性比效率更重要
- 使用原子設計(Atomic Design)設計程式

### UNIX 哲學

- 小即是美
- 讓程式只做好一件事
- 儘可能早地建立原型
- 沉默是金（減少不必要的日誌輸出）
- 並列地思考
- 部分加部分大於整體

## 命名規則

- 變數：使用有意義的小寫蛇形命名（snake_case）或駝峰命名（camelCase，視語言而定）
- 函數：動詞開頭，描述行為
- 類別：名詞，描述實體

## 錯誤處理

- 明確的錯誤訊息
- 使用適當的錯誤類型
- 記錄關鍵錯誤但避免過度日誌

## API 設計原則

- graphql 來傳輸資料
- 可以參考後端程式碼 https://github.com/jeby-228/mantra_API

## 使用套件庫

- 使用成熟且廣泛使用的套件庫
- 定期更新依賴以獲得安全修復和新功能
- 非必要不要撰寫 CSS 樣式，盡量使用現有的 UI 框架或主題

---
> Source: [jeby-228/mantra_API_frontend](https://github.com/jeby-228/mantra_API_frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
