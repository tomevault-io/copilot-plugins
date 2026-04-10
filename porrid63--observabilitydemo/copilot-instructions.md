## observabilitydemo

> 本專案是一個可觀測性（Observability）綜合演示專案，涵蓋日誌、指標與追蹤。採用 4 個獨立 .NET 10.0 微服務專案，透過 **gRPC** 做同步通訊、**Kafka** 做非同步 event-driven，產生真實的分散式遙測數據，並整合 Seq、ELK Stack 及 Grafana Stack 等工具進行收集與視覺化。

# GUIDE.md — 遊戲平台可觀測性展示專案 (Game Platform Observability Demo)

## 專案概述

本專案是一個可觀測性（Observability）綜合演示專案，涵蓋日誌、指標與追蹤。採用 4 個獨立 .NET 10.0 微服務專案，透過 **gRPC** 做同步通訊、**Kafka** 做非同步 event-driven，產生真實的分散式遙測數據，並整合 Seq、ELK Stack 及 Grafana Stack 等工具進行收集與視覺化。

### 微服務架構

| 專案 | 包含服務 | 角色 | 埠號 |
|------|---------|------|------|
| **GatewayService** | ApiGateway | Workflow 編排器，透過 gRPC 呼叫下游 | 5100 |
| **PlayerGameService** | PlayerService + GameService | gRPC server，玩家登入/驗證 + 遊戲操作 | 5200 |
| **FinanceService** | WalletService + PaymentService + RiskService | gRPC server，錢包/支付/風控 + Kafka producer | 5300 |
| **NotificationService** | NotificationService | Worker SDK，純 Kafka consumer | — |

### 通訊模式

**gRPC (同步):**
- Gateway → PlayerGameService: 登入、驗證、遊戲開始、下注、遊戲結果
- Gateway → FinanceService: 存款、提款、結算、餘額更新
- PlayerGameService → FinanceService: 餘額查詢

**Kafka (非同步):**
- `bet-settled` → NotificationService
- `payment-processed` → NotificationService
- `withdrawal-approved` / `withdrawal-flagged` → NotificationService
- `workflow-completed` → NotificationService
- `insufficient-balance` → NotificationService

---

## 專案結構

```
ObservabilityDemo/
├── ObservabilityDemo.sln
├── protos/                            # 共享 .proto 定義
│   ├── player_game.proto              # PlayerService、GameService RPC 定義
│   └── finance.proto                  # WalletService、PaymentService、RiskService RPC 定義
├── src/
│   ├── ObservabilityDemo.Shared/                # 共享：OTel 設定、Kafka 工具、Event DTOs、常數
│   ├── ObservabilityDemo.GatewayService/        # Port 5100, BackgroundService + gRPC client
│   ├── ObservabilityDemo.PlayerGameService/     # Port 5200, gRPC server
│   ├── ObservabilityDemo.FinanceService/        # Port 5300, gRPC server + Kafka producer
│   └── ObservabilityDemo.NotificationService/   # Worker SDK, 純 Kafka consumer
├── dotnet-demo/                       # 保留原始單體版本作為參考
├── config/                            # 各元件設定（OTel Collector、Grafana、Loki、Tempo、Prometheus、Elasticsearch）
├── docker-compose.yml                 # 可觀測性基礎設施 + Kafka (KRaft)
├── start-all.bat                      # Windows 快速啟動腳本
└── start-all.sh                       # Linux/Mac 快速啟動腳本
```

---

## 技術堆疊

### 應用程式端
- **語言：** C# / .NET 10.0
- **日誌：** Serilog 4.3 搭配 OpenTelemetry sink，透過 OTLP 匯出
- **追蹤：** OpenTelemetry 1.15（每個微服務各一個 `ActivitySource`）
- **同步通訊：** gRPC (Grpc.Net.Client 2.76.0 / Grpc.AspNetCore)
- **非同步通訊：** Kafka (Confluent.Kafka 2.13.0)
- **Trace 傳播：** gRPC 自動 (OTel Instrumentation)、Kafka 手動 (W3C traceparent in headers)
- **傳輸協定：** OTLP (gRPC) 至 Collector

### 基礎設施（Docker Compose）
- **OpenTelemetry Collector** contrib 0.118.0 — 接收並轉發遙測數據
- **Seq** — 結構化日誌查詢與分析
- **Grafana** — 統一儀表板（整合 Loki、Tempo、Prometheus）
- **Loki** — 日誌聚合後端
- **Tempo** — 分散式追蹤後端
- **Prometheus** — 指標後端
- **Elasticsearch** — 搜尋與分析引擎
- **Kibana** — Elasticsearch 視覺化介面
- **Kafka** (KRaft mode, confluentinc/cp-kafka:7.8.0) — 非同步訊息佇列
- **Kafka UI** (provectuslabs/kafka-ui) — Kafka 管理介面

---

## 資料流架構

```
GatewayService ──gRPC──▶ PlayerGameService ──gRPC──▶ FinanceService
      │                                                    │
      │                                              Kafka produce
      │                                                    │
      └──────── Kafka produce ──────────────▶ NotificationService (Kafka consume)

所有服務 ──OTLP/gRPC :4317──▶ OTel Collector ──▶ Seq / Loki / Tempo / Prometheus / Elasticsearch
```

### 服務埠號

| 服務 | 埠號 | 協定 |
|------|------|------|
| GatewayService | 5100 | gRPC client only |
| PlayerGameService | 5200 | gRPC server |
| FinanceService | 5300 | gRPC server |
| NotificationService | — | Kafka consumer only |
| Kafka | 9092 | Kafka protocol |
| Kafka UI | 8080 | HTTP |
| OTel Collector | 4317 (gRPC)、4318 (HTTP) | OTLP |
| Seq | 5341 | HTTP |
| Grafana | 3000 | HTTP |
| Prometheus | 9090 | HTTP |
| Loki | 3100 | HTTP |
| Tempo | 3200 | HTTP |
| Elasticsearch | 9200 | HTTP |
| Kibana | 5601 | HTTP |

---

## 建置與執行

### 快速啟動

```bash
# Windows
start-all.bat

# Linux / Mac
./start-all.sh
```

### 分步操作

```bash
# 1. 啟動基礎設施（等待約 30 秒讓所有服務初始化完成）
docker compose up -d

# 2. 建置整個解決方案
dotnet build ObservabilityDemo.sln

# 3. 依序啟動 4 個微服務（各開一個終端）
cd src/ObservabilityDemo.FinanceService && dotnet run         # 先啟動 gRPC server
cd src/ObservabilityDemo.PlayerGameService && dotnet run       # 再啟動 gRPC server
cd src/ObservabilityDemo.GatewayService && dotnet run          # 啟動 workflow 編排器
cd src/ObservabilityDemo.NotificationService && dotnet run     # 啟動 Kafka consumer

# 4. 關閉服務堆疊
docker compose down          # 保留 volumes
docker compose down -v       # 移除 volumes

# 5. 檢視特定服務日誌
docker compose logs <service>
```

### 原始單體版本

```bash
cd dotnet-demo && dotnet run
```

---

## 模擬業務邏輯（Workflows）

應用程式持續產生以下三種業務流程，每個工作流程以 2 秒間隔循環執行，並包含機率性的錯誤/警告注入。可透過 `WorkflowName` 進行過濾。

| 工作流程            | 步驟數 | 說明                                               |
|--------------------|--------|---------------------------------------------------|
| **BettingWorkflow**    | 8      | 登入、驗證、餘額檢查、遊戲開始、下注、結果、結算、餘額更新 |
| **DepositWorkflow**    | 4      | 發起、驗證、處理、入帳                               |
| **WithdrawalWorkflow** | 3      | 請求、風險評估、核准/人工審核                         |

錯誤情境的完整目錄、觸發機率與結構化資料 schema 詳見 `dotnet-demo/ERROR_AND_WARNING_SCENARIOS.md`。

---

## 遙測模式

- Serilog 搭配 OpenTelemetry sink，透過 OTLP 匯出日誌
- 每個微服務各有獨立的 `ActivitySource`，用於分散式追蹤
- gRPC 通訊：OTel Instrumentation 自動傳播 W3C traceparent（零配置）
- Kafka 通訊：手動注入/提取 traceparent 到 message headers，用 `parentContext` 接續同一 Trace
- 透過 `LogContext` 推送結構化日誌屬性（TraceId、CorrelationId、UserId、SessionId）
- 豐富的巢狀物件（LoginContext、BetDetails、RiskAssessment 等）作為結構化日誌資料

### TraceId 傳播

同一個 Workflow 的所有操作（跨 4 個 process、跨 gRPC + Kafka）共用同一個 TraceId。在 Tempo/Seq 中查詢一個 TraceId 即可看到完整的 span 鏈。

### 日誌分析常用欄位

| 欄位             | 說明                                            |
|-----------------|------------------------------------------------|
| `TraceId`       | 追蹤單次完整 Workflow 的唯一標識                    |
| `CorrelationId` | 關聯相關聯的多個 Workflow                          |
| `UserId`        | 模擬玩家 ID（例如 `USER_123`）                     |
| `SessionId`     | 會話 ID                                          |
| `WorkflowName`  | 工作流程名稱                                      |
| `WorkflowStep`  | 具體流程步驟（例如 `5-PlaceBet`）                   |
| `service.name`  | 各服務獨立的 service.name                          |

**查詢範例：** 若要尋找特定使用者的下注歷史，請過濾 `UserId` 並篩選 `WorkflowName = 'BettingWorkflow'`。

---

## 程式碼風格與規範

- C# 啟用可為 null 的參考型別與 implicit usings
- 4 空白縮排
- 公開型別與成員使用 `PascalCase`，區域變數與參數使用 `camelCase`
- 共用邏輯放在 `ObservabilityDemo.Shared` 專案中
- Proto 定義放在 `protos/` 目錄
- 變更應聚焦且最小化

---

## 測試指引

- 目前沒有自動化測試
- 若新增測試，建議建立 `tests/ObservabilityDemo.Tests`，並以 `dotnet test` 執行
- 測試類別以 `*Tests` 結尾，測試資料與情境保持就近放置

---

## Commit 與 Pull Request 規範

### Commit 格式
```
<emoji> <type>(scope): summary
```
範例：`✨ feat(readme): update docs`

建議使用的 type：`feat`、`refactor`、`chore`，以簡短命令式摘要描述變更。

### Pull Request 要求
- 說明目的與行為變更
- 描述如何執行與測試狀態
- 若修改可觀測性設定，需說明影響的服務與驗證方式

---

## 安全與設定注意事項

- 預設端點以 `docker-compose.yml` 為準，避免硬編碼機密
- 新增環境變數時，請同步更新 `README.md`

---

## 故障排除

- **基礎設施未啟動：** 檢查 `docker logs <container_name>`
- **應用程式連線失敗：** 確保程式能成功連接到 `localhost:4317`（OTel Collector）
- **Kafka 連線失敗：** 確保 Kafka 在 `localhost:9092` 運行中
- **gRPC 連線失敗：** 確保 FinanceService (5300) 和 PlayerGameService (5200) 先於 GatewayService 啟動
- **服務初始化：** 啟動基礎設施後，請等待約 30 秒讓所有服務初始化完成

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PorriD63) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
