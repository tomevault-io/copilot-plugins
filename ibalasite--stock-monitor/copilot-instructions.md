## stock-monitor

> 最後更新：2026-04-14（Asia/Taipei, v1.0）

# CLAUDE.md - Stock Monitor AI 開發手冊

最後更新：2026-04-14（Asia/Taipei, v1.0）
對齊文件：`PDD_Stock_Monitoring_System.md`、`EDD_Stock_Monitoring_System.md`、`TEST_PLAN.md`、`USER_STORY_ACCEPTANCE_CRITERIA.md`、`API_CONTRACT.md`、`ADR.md`、`NFR_SLI_SLO.md`、`SECURITY_AND_SECRETS.md`、`OPERATIONS_RUNBOOK.md`

本檔與 `CODEX.md` 保持同一份規格語意，任一檔更新時需同步另一檔。

## 0. 使用方式
本檔是 Claude 進入專案時的執行基線。開始任何修改前，先依本檔確認：
1. 規格優先順序
2. 不可變更的業務規則
3. tests 要求的 symbol contract

## 1. 規格優先順序（衝突時依序採用）
1. `features/stock_monitoring_system.feature` + `tests/*.py`（可執行規格）
2. `EDD_Stock_Monitoring_System.md`
3. `PDD_Stock_Monitoring_System.md`
4. `USER_STORY_ACCEPTANCE_CRITERIA.md`

已知衝突已定版：
- `idempotency_key` 以 `stock_no + minute_bucket`（不含 `stock_status`）為準。
- 冷卻鍵仍是 `stock_no + stock_status`。

## 2. 產品一句話
台股盤中每分鐘監控 + LINE 群組彙總通知 + 每交易日 14:00 估值入 SQLite + 5 分鐘防洗版，且能擴充多估值方法。

## 3. 不可變更常數
| 項目 | 值 |
|---|---|
| 時區 | `Asia/Taipei` |
| 輪詢間隔 | `60` 秒 |
| 冷卻秒數 | `300` 秒 |
| 冷卻維度 | `stock_no + stock_status` |
| 交易時段 | `09:00-13:30`（含） |
| 開盤檢查起始 | `08:45` |
| 不開市判定 | `09:00` 後仍無大盤當日新資料 |
| 報價新鮮度門檻 | `STALE_THRESHOLD_SEC=90` |
| 行情重試上限 | `MAX_RETRY_COUNT=3` |
| 每分鐘 LINE 訊息數 | 最多 `1` 封（彙總） |
| 日結估值時間 | `14:00` |

## 4. LINE 設定與 fail-fast 規則
Canonical 名稱：
- `LINE_CHANNEL_ACCESS_TOKEN`
- `LINE_TO_GROUP_ID`

Alias（等效）：
- `CHANNEL_ACCESS_TOKEN`
- `TARGET_GROUP_ID`

必須通過：
- 缺任一必要值 -> fail-fast
- token/group id 格式明顯錯誤 -> fail-fast
- 錯誤訊息與 log 不得出現明文 token- **CR-SEC-01**：`LinePushClient` 持有物件不得透過 `repr()` 或 log 輸出 token 明文；`field(repr=False)` 或等效保護為庞就需求
## 5. 核心業務規則
### 5.1 訊號規則
- `status=1`：`market_price <= fair_price`
- `status=2`：`market_price <= cheap_price`
- 同分鐘同股票同時命中 1/2 -> 只保留 2

### 5.2 多方法聚合
- 同分鐘同股票，多方法都命中 status 1 -> 只產生一個股票事件
- 若同時有方法命中 status 1 與 status 2 -> 統一輸出 status 2
- `methods_hit` 必須保留全部命中方法

### 5.3 冷卻 + 冪等
- 冷卻判斷：查 `message(stock_no, stock_status)` 最近 `update_time`
- `last_sent_at IS NULL` -> 可發送
- 5 分鐘內同冷卻鍵 -> 不發送、且不更新 `message.update_time`
- 同分鐘冪等鍵：`{stock_no}|{minute_bucket}`

### 5.4 每分鐘訊息與補償
- 同分鐘所有股票合併成一封 LINE 訊息發送
- 所有出站 LINE 訊息（彙總/摘要/觸發列/測試推播）都必須透過 `template_key + context` 渲染
- LINE 失敗：不寫 `message`，寫 `system_logs`
- LINE 成功 + DB 寫入失敗：寫 `pending_delivery_ledger`，若 DB 不可寫則 fallback `logs/pending_delivery.jsonl`
- 補償期間視為已通知，避免重複發送

### 5.5 跳過分鐘不得補發
下列情境該分鐘都不得補發過期訊號：
- `MARKET_TIMEOUT`
- `STALE_QUOTE`
- `DATA_CONFLICT`
- 重試耗盡（`TP-INT-011`）

## 6. 資料模型最小要求
- `watchlist`：`manual_cheap_price <= manual_fair_price`
- `valuation_methods`：同 `method_name` 僅允許一個 `enabled=1`
- `valuation_snapshots`：唯一鍵必含 `method_version`
- `message`：`UNIQUE(stock_no, minute_bucket)`
- `methods_hit`：需為 JSON；實作建議加嚴為 JSON array（`json_type='array'`）
- `pending_delivery_ledger.status`：`PENDING | RECONCILED | FAILED`

## 7. 測試要求的 Python symbol contract
| 模組 | Symbol |
|---|---|
| `stock_monitor.db.schema` | `SCHEMA_SQL` |
| `stock_monitor.bootstrap.runtime` | `assert_sqlite_prerequisites`, `validate_line_runtime_config` |
| `stock_monitor.bootstrap.health` | `health_check` |
| `stock_monitor.domain.policies` | `PriorityPolicy`, `CooldownPolicy`, `aggregate_stock_signals` |
| `stock_monitor.domain.idempotency` | `build_minute_idempotency_key` |
| `stock_monitor.domain.time_bucket` | `TimeBucketService`, `guard_bucket_source` |
| `stock_monitor.domain.metrics` | `compute_notification_accuracy` |
| `stock_monitor.application.message_template` | `render_line_template_message`（唐一定義來源，CR-ARCH-03） |
| `stock_monitor.application.monitoring_workflow` | `aggregate_minute_notifications`, `merge_minute_message`, `dispatch_and_persist_minute`, `reconcile_pending_once`, `guard_minute_execution`, `persist_message_rows_transactional`, `fetch_market_with_retry` |
| `stock_monitor.application.trading_session` | `evaluate_market_open_status`, `is_in_trading_session` |
| `stock_monitor.application.valuation_scheduler` | `run_daily_valuation_job` |
| `stock_monitor.application.valuation_calculator` | `ManualValuationCalculator` |
| `stock_monitor.application.runtime_service` | `MinuteCycleConfig` |
| `stock_monitor.uat.scenarios` | `UAT_SCENARIOS` |
| `stock_monitor.adapters.market_data_twse` | `TwseRealtimeMarketDataProvider`（含 `_price_cache`、`_exchange_cache`、`_tick_cache`） |
| `stock_monitor.adapters.market_data_yahoo` | `YahooFinanceMarketDataProvider` |
| `stock_monitor.adapters.market_data_composite` | `CompositeMarketDataProvider` |

## 8. TDD 執行規範
1. 先跑對應測試確認紅燈
2. 寫最少主程式讓測試轉綠
3. 重構但不破壞測試
4. 每個需求點小步提交

常用指令：
```bash
python -m pytest -q tests
python -m pytest -q tests/test_policy_rules.py
python -m pytest -q tests/test_integration_workflow.py -k TP-INT-010
```

## 9. 建議實作順序
1. `db/schema.py`
2. `domain/policies.py`
3. `domain/idempotency.py`
4. `domain/time_bucket.py`
5. `domain/metrics.py`
6. `bootstrap/runtime.py` + `bootstrap/health.py`
7. `application/trading_session.py`
8. `application/monitoring_workflow.py`
9. `application/valuation_scheduler.py`
10. `uat/scenarios.py`

## 10. 完成定義（DoD）
- `TP-DB-*`, `TP-ENV-*`, `TP-POL-*`, `TP-INT-*`, `TP-TRD-*`, `TP-VAL-*`, `TP-UAT-*` 全部綠燈
- `TP-SEC-*`, `TP-ARCH-*`, `TP-ADP-*` 全部綠燈（Code Review 改善項目）
- UAT 14 條可追溯
- coverage gate：`lines/branches/functions/statements = 100%`
- 文件同步：若規則有變更，`PDD/EDD/TEST_PLAN/feature/CLAUDE/CODEX` 一並更新

## 11. Code Review 改善禁止清單（v1.0 定版，業務程式禁寫）
- **禁止** `scenario_case` 分支存在於任何生產估値計算路徑（CR-SEC-02、CR-ARCH-02）
- **禁止** `_resolve_timezone` 對無效時區名稱靜默 fallback UTC（CR-SEC-03）
- **禁止** 在 `message_template.py` 以外的模組重複定義 `render_line_template_message`（CR-ARCH-03）
- **禁止** 在 `app.py`（CLI Interface Layer）內定義估値計算邏輯（CR-ARCH-01）
- **禁止** `TimeBucketService` 對無效時區名稱靜默設 `self._tz = None`（CR-CODE-05）
- **禁止** `YahooFinanceMarketDataProvider` 在 HTTP 失敗時 raise exception 向上傳播（CR-ADP-01）
- **禁止** `CompositeMarketDataProvider` 直接回傳任一來源的字典而不做 Freshness-First 比較（CR-ADP-02）

## 12. Out of Scope（此階段不做）
- 自動下單
- 多市場（美股/加密）
- 分散式高可用部署

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ibalasite) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
