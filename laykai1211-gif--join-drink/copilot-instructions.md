## join-drink

> > 本文件摘要自 `API.md` 與 `DATABASE.md`，供 Gemini CLI 快速掌握專案背景。

# JoinDrink 專案指南

> 本文件摘要自 `API.md` 與 `DATABASE.md`，供 Gemini CLI 快速掌握專案背景。

## 🚀 專案概述
JoinDrink 是一個飲品訂購平台，支援個人點餐、揪團訂購、品牌/分店管理以及錢包與優惠券系統。

## 🛠 核心技術棧
- **後端**: Java 17, Spring Boot 3.4.3
- **資料庫**: MySQL (Aiven Cloud), Spring Data JPA (Hibernate)
- **認證**: JWT (JJWT), Spring Security, Firebase Admin SDK (第三方登入)
- **圖片處理**: Cloudinary SDK
- **命名規範**: 資料庫使用 `snake_case`，主鍵為 `id` (BIGINT AUTO_INCREMENT)。

## 🏗 系統架構與角色
系統基於角色存取控制 (RBAC)，主要分為三種角色：
1. **CUSTOMER (一般用戶)**: 搜尋門市、點餐、發起/加入揪團、管理錢包與優惠券。
2. **BRAND (品牌管理)**: 管理菜單分類、飲品模板、規格(Spec)、配料(Topping)及區域加價設定。
3. **STORE (分店運營)**: 管理訂單狀態、營業時間、外送設定、商品售罄狀態。

## 💡 關鍵功能模組
- **揪團訂購 (Group Order)**: 基於 Token 分享，支援多人加入、團長提交、代管扣款。
- **錢包系統 (Wallet)**: 支援儲值、代管 (Escrow)、退款、結算，完整交易紀錄追蹤。
- **區域加價設定**: 品牌可針對不同地區 (如北/中/南) 設定分類加價位移 (Price Offset)。
- **動態規格與配料**: 支援平台預設與品牌自訂規格/配料，分店可獨立切換供應狀態。
- **轉盤抽獎 (Spin Wheel)**: 每日抽獎獲取優惠券，結合 Java 2D 動態合成圖片與 Cloudinary。

## 📍 開發上下文
- **Base URL**: `http://localhost:8082`
- **API 統一回應格式**:
  ```json
  { "code": 200, "msg": "success", "data": { ... } }
  ```
- **訂單狀態流**: `OPEN` → `SUBMITTED` → `READY` → `COMPLETED`。

## 📝 交流與規範
- **語言要求**: 所有對話、代碼註解、錯誤分析及功能說明，請一律使用**繁體中文**。即使收到英文詢問，亦須以繁體中文回答。

## 📂 重要文件參考
- `API.md`: 完整的 RESTful API 端點與參數定義。
- `DATABASE.md`: 詳細的資料表結構 (Table Schema) 與關聯圖 (ER Diagram)。
- `pom.xml`: 專案依賴與建構組態。

測試連線

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/laykai1211-gif) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
