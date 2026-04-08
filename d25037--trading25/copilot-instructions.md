## trading25

> あなたは apps/bt/ と apps/ts/ の結合を統合管理するオーケストレーターです。

## Role
あなたは apps/bt/ と apps/ts/ の結合を統合管理するオーケストレーターです。
subagentsを用いてそれぞれのプロジェクトを横断的に把握します。

## データフロー・ポート割り当て

```
JQUANTS API ──→ FastAPI (:3002) ──→ Data Plane
                     │               ├─ DuckDB + Parquet (market time-series)
                     │               ├─ DuckDB + Parquet (dataset snapshots)
                     │               └─ SQLite (portfolio/jobs metadata)
                     ↓
                  ts/web (:5173)
                  ts/cli
```

| サービス | ポート | 技術 |
|---|---|---|
| bt/server | 3002 | FastAPI + uvicorn |
| ts/web | 5173 | Vite + React 19 |

- **FastAPI** が唯一のバックエンド
- **bt** はデータ種別ごとに DuckDB / SQLite に直接アクセス（`contracts/` スキーマ準拠）
  - **market.duckdb + parquet**: market time-series / metadata 読み書き（DuckDB SoT）
- **datasets/<name>/**: `dataset.duckdb + parquet/ + manifest.v2.json` のみをサポートする snapshot bundle
- **portfolio.db**: portfolio/watchlist CRUD + jobs metadata 永続化（SQLAlchemy Core）
- market 時系列 Data Plane の SoT は DuckDB + Parquet のみ（SQLite mirror は廃止）
- `stock_data_raw` は raw `O/H/L/C/Vo + AdjFactor` の SoT、`stock_data` は local projection で生成した adjusted series とする
- `POST /api/db/sync` の `dataPlane` override は `backend=duckdb-parquet` のみ受け付ける
- `GET /api/db/stats` / `GET /api/db/validate` の時系列スナップショット SoT は DuckDB inspection（`timeSeriesSource` を返却）で、`margin_data` と `options_225_data` の件数・日付範囲・diagnostics もここから返す。`/api/db/validate` は deferred な fundamentals/margin empty-response negative cache を info 扱いに留め、TOPIX に対して `options_225_data` が未投入・stale・partial coverage の場合は actionable warning として返す
- `market.duckdb` の `incremental sync` は `topix_data` / `stock_data` / `indices_data` / `margin_data` / `options_225_data` を更新する。`index_master` はローカル catalog を SoT として補完し、`indices_data` は code 指定同期（catalog + 既存DBコード）を基本に、日付指定同期で新規コードを補完する。不足 `index_master` はプレースホルダ補完し、FK 制約付きの既存DBでも継続可能にする。日経225オプション同期時は `UnderPx` が日付単位で単一値のときだけ synthetic index `N225_UNDERPX` を生成する
- `/api/db/sync`（`initial` / `incremental` / `repair`）は `Bulk+REST hybrid` を stage 単位で選択し、予測外部request数が最小の手法を優先する。`stock_data` stage は `enforceBulkForStockData=true` の場合に bulk 必須とし、bulk 不可時は REST fallback せず理由付きでエラー終了する。incremental `stock_data` は TOPIX の新規日付だけでなく DuckDB inspection の `missing_stock_dates` も backfill 対象に含める。`margin_data` stage も `/markets/margin-interest` で bulk/rest を選択し、既存 `market.duckdb` に margin snapshot が無い場合は `incremental` でも full backfill する。`options_225_data` stage は `/derivatives/bars/daily/options/225` を日付単位で同期し、`initial` は `topix_data` 日付集合を基準に backfill、`incremental` は latest options anchor 以降を対象にするが、options anchor が無く local TOPIX がある場合は full local `topix_data` 日付集合で backfill する。margin backfill 対象は current listed markets（`0111/0112/0113`）のみとし、empty response は trading frontier 単位の negative cache として suppress する。fundamentals backfill は preferred-share listed code を親 issuer code へ canonicalize し、empty response は disclosed frontier 単位の negative cache として suppress する。stock ingest は raw `O/H/L/C/Vo + AdjFactor` を `stock_data_raw` に保存し、`adjustment_factor != 1` を含む code は local projection で `stock_data` を再生成する。`incremental` で DuckDB inspection の `topix/stock/indices` が空かつアンカー不在なら cold-start bootstrap を選び、全日付 fallback 暴走を回避する。`repair` は listed-market fundamentals backfill など非 price warning の回復に限定し、adjustment refresh は行わない。`stocksNeedingRefresh` は互換フィールドとして残すが常時 0 / empty を返す
- `/api/db/sync/jobs/active` は実行中ジョブの再取得 SoT、`/api/db/sync/jobs/{jobId}/fetch-details` は fetch strategy/execution 詳細の SoT とし、web は再読み込み/再訪時に localStorage と組み合わせて active job 追跡を復元する。`/api/db/sync/jobs/{jobId}` が 404 の場合は stale な jobId を破棄する
- `stock_data` の bulk ingest は bulk file 単位で publish し、大量期間同期時のメモリピークによる bulk 失敗→REST fallback を抑制する。fallback 時の progress message は reason を含める
- legacy `market.duckdb`（`stock_price_adjustment_mode` 無し、または `stock_data_raw` 不在で `stock_data` のみ存在）は incompatible とし、`initial` sync の `resetBeforeSync=true` で `market-timeseries/market.duckdb` と `market-timeseries/parquet/` を reset して再構築するまで `auto` / `incremental` / `repair` / `stocks/refresh` を拒否する
- `/api/db/sync` と `POST /api/db/stocks/refresh` の時系列アンカー判定・publish/index は DuckDB inspection + time-series store を必須 SoT とし、旧 SQLite 時系列テーブルへの fallback を禁止する（inspection 失敗時はエラーで停止）。`POST /api/db/stocks/refresh` は adjusted 値の再取得ではなく raw refresh + local reprojection を行う
- DuckDB time-series store は `publish/index/inspect/close` をプロセス内ロックで直列化し、sync 実行と `/api/db/stats` `/api/db/validate` 参照の同時実行による 500 を防止する
- DuckDB time-series store の large-batch publish（`stock_data` / `indices_data` / `margin_data` / `options_225_data`）は relation-based upsert を使い、`executemany` ボトルネックによる sync timeout を避ける
- DuckDB time-series store の Parquet export は `stock_data` / `indices_data` で全件 `ORDER BY` を行わず、同期スループットを優先する（row order 非依存を前提とする）
- DuckDB `topix_data` は `open=high=low=close` かつ `open=前日close` の異常日足を除外する（取込時 + 起動時クリーンアップ）
- `market.duckdb` の `statements` upsert は `(code, disclosed_date)` 衝突時に非NULL優先マージ（`coalesce(excluded, existing)`）とし、同日別ドキュメント取り込み時の forecast 欠損上書きを防止する
- Backtest 実行パスは `BT_DATA_ACCESS_MODE=direct` で `DatasetSnapshotReader` / `MarketDbReader` を直接参照し、FastAPI 内部HTTPを経由しない
- Dataset snapshot は `dataset.duckdb` と `manifest.v2.json` の両方が揃った bundle のみ解決対象とし、旧 `dataset.db` / root-level `*.db` snapshot は unsupported とする
- Dataset API `GET /api/dataset/{name}/info` の SoT は `snapshot` + `stats` + `validation`（`details.dataCoverage` / `details.fkIntegrity` / `details.stockCountValidation` 含む）で、DuckDB snapshot のみを返す
- Dataset create（`POST /api/dataset`）は `market.duckdb` snapshot を SoT とし、`stock_data` / `statements` / `margin_data` / `topix_data` / `indices_data` を DuckDB 直結 copy で取り込む。`stock_data` は 4桁/5桁 code を正規化し、同日重複は 4桁優先 + 5桁 NULL backfill で 1 行化し、欠損OHLCV row の warning 集約と partial snapshot / manifest 挙動を維持する。生成物は `dataset.duckdb + parquet/ + manifest.v2.json`。直結 source が使えない場合のみ legacy query path へ fallback する。再試行は `overwrite=true` に統一し、`resume` route と `timeoutMinutes` request override は廃止した。backend job timeout は内部固定値のみを使う
- Dataset cancel は API caller に対して即時返却し、cancelled build は `manifest.v2.json` を書かない partial snapshot のまま resolve 不可にする。in-flight cleanup/export は background task 完了まで継続しうる
- Backtest result summary の SoT は成果物セット（`result.html` + `*.metrics.json`）。`/api/backtest/jobs/{id}` と `/api/backtest/result/{id}` は成果物から再解決し、必要時のみ job memory/raw_result をフォールバックとして使う
- Screening API は非同期ジョブ方式（`POST /api/analytics/screening/jobs` / `GET /api/analytics/screening/jobs/{id}` / `POST /api/analytics/screening/jobs/{id}/cancel` / `GET /api/analytics/screening/result/{id}`）を SoT とする。旧 `GET /api/analytics/screening` は 410
- Screening 実行時のデータ SoT は `market.duckdb`（`stock_data` / `topix_data` / `indices_data` / `stocks` / `margin_data`）とし、dataset へのフォールバックを禁止する
- `Charts`/`Analysis` の `stock_data` 読み取りは 4桁/5桁コード混在を正規化し、同日重複行は 4桁コード優先で 1 行化する。`stocks` 欠落時でも `stock_data` からの OHLCV 取得を継続する
- Screening / Charts / Backtest / Signal semantics の SoT matrix は [`docs/architecture-sot-matrix.md`](docs/architecture-sot-matrix.md) を参照する
- Strategy 設定検証の SoT は backend strict validation（`/api/strategies/{name}/validate` と保存時検証）で、frontend のローカル検証は補助扱い（deprecated）。`production/*` は raw YAML で `shared_config.dataset` の明示宣言を必須とし、`default.yaml` 継承のみでは不十分とする
- Backtest family（`backtest` / `attribution` / `optimize` / `lab`）は `shared_config.execution_policy.mode` として `next_session_round_trip` / `current_session_round_trip` / `overnight_round_trip` をサポートする。`next_session_round_trip` は entry signal の翌営業日 `Open` で建てて同日 `Close` で閉じ、`current_session_round_trip` は当日 `Open` で建てて当日 `Close` で閉じ、`overnight_round_trip` は当日 `Close` で建てて翌営業日 `Open` で閉じる。`screening` は `next_session_round_trip` を unsupported error で拒否し、production strategy を `screening_support` (`supported` / `unsupported`) と `entry_decidability` (`pre_open_decidable` / `requires_same_session_observation`) で分類する
- Strategy YAML更新の SoT は `/api/strategies/{name}` で、`production` / `experimental` を更新可能（`production` は既存ファイルの編集のみ許可）。`rename` / `delete` は引き続き `experimental` 限定
- Strategy `rename` / `delete` の権限判定はトップレベルカテゴリ基準で行い、`experimental/**`（例: `experimental/optuna/foo`）は許可する
- 市場コードフィルタは legacy (`prime/standard/growth`) と current (`0111/0112/0113`) を同義として扱う
- **ts/web** は `/api` パスを FastAPI (:3002) にプロキシ
- **Hono サーバー** (:3001) は廃止済み（`apps/ts/packages/api` は削除済み）
- `GET /api/market/options/225` は DuckDB local read を SoT とし、`options_225_data` が空なら `market_db_sync` recovery を返して live proxy fallback は行わない

## OpenAPI契約

bt が FastAPI の OpenAPI スキーマを公開し、ts/contracts が型を自動生成する。
```bash
bun run --filter @trading25/contracts bt:sync   # bt の OpenAPI → TS型生成
```
スキーマ変更時は必ず `bt:sync` を実行し、`contracts/` 配下も更新すること。
- `apps/ts` workspace は `@redocly/openapi-core` を `1.34.5` に固定し、`bt:sync`（openapi-typescript）を安定実行する

## contracts/ ガバナンス

`contracts/` に bt/ts 間の安定インターフェースを定義。詳細は [`contracts/README.md`](contracts/README.md) 参照。
- **バージョニング**: additive (minor) / breaking (major) → 新版ファイル作成
- **命名規則**: `{domain}-{purpose}-v{N}.schema.json`
- **現行追加契約**: `fundamentals-metrics-v2.schema.json`（fundamentals API 指標拡張、`bookToMarket` を削除）
- **portfolio DB 契約 SoT**: `portfolio-db-schema-v2.json`（`jobs` テーブルを含む）
- **アーカイブ**: `hono-openapi-baseline.json`（Phase 3 移行 baseline、参照用に保持）

## エラーレスポンス

統一エラーレスポンスフォーマット:
```json
{"status":"error","error":"Not Found","message":"...","details?":[...],"timestamp":"...","correlationId":"..."}
```
- FastAPI: 例外ハンドラが `HTTPException(detail=...)` を自動変換
- `RequestLoggerMiddleware` が `JQuantsApiError`(502/504) / `SQLAlchemyError`(500) / 汎用例外(500) をキャッチし統一フォーマットで返却
- correlation ID: `x-correlation-id` ヘッダで伝播（なければ自動生成）
- 内部HTTP呼び出し（`src/api/client.py`）も `x-correlation-id` を伝播
- ErrorResponse スキーマは OpenAPI で全エンドポイントに 400/404/500 として公開

## J-Quants Proxy キャッシュ/観測

- `JQuantsProxyService` は in-memory TTL + singleflight を使用
  - `/markets/margin-interest`: 5分
  - `/fins/summary`（`/statements` / `/statements/raw` で共有）: 15分
  - `/derivatives/bars/daily/options/225`: 5分。`date` 未指定時の latest resolution も同TTLで共有する
- 実外部呼び出しは `event="jquants_fetch"`、キャッシュ状態は `event="jquants_proxy_cache"` で構造化ログ出力
- market sync の fetch planner は `event="sync_fetch_strategy"` を出力し、各 stage の選択結果（bulk/rest, 推定request数, 実request数, cache hit/miss）を観測可能にする

## ミドルウェア構成（FastAPI）

登録順（LIFO: 下から上に実行）:
1. **RequestLoggerMiddleware** — リクエストロギング（最外側）
2. **CorrelationIdMiddleware** — correlation ID 管理
3. **CORSMiddleware** — CORS（最内側）

- OpenAPI 設定は `openapi_config.py` に集中管理
- ドキュメント UI: `/doc`（Swagger UI）、`/docs` `/redoc` は無効

## 共有XDGパス

両プロジェクトが `~/.local/share/trading25/` を共有:
- `market-timeseries/market.duckdb` + `datasets/` + `portfolio.db` — FastAPI が管理
- `strategies/experimental/` / `strategies/production/` / `strategies/legacy/` / `backtest/results/` / `backtest/attribution/` — bt が管理
- `config/default.yaml` は repo baseline（`apps/bt/config/default.yaml`）を残しつつ、`shared_config.dataset` は空欄 baseline とする。runtime override は `TRADING25_DEFAULT_CONFIG_PATH` または `~/.local/share/trading25/config/default.yaml` を優先する

## bt (Python / uv)
VectorBT基盤の高速バックテスト・Marimo Notebook実行システム。
FastAPI サーバー（:3002）とtyper CLI を提供。

```bash
uv sync                          # 環境セットアップ
uv run bt server --port 3002     # APIサーバー起動
uv run bt backtest <strategy>    # バックテスト実行
uv run bt lab generate --entry-filter-only --allowed-category fundamental
uv run bt lab evolve <strategy> --entry-filter-only --allowed-category fundamental
uv run bt lab optimize <strategy> --entry-filter-only --allowed-category fundamental
uv run bt lab improve <strategy> --entry-filter-only --allowed-category fundamental
uv run bt migrate-optimization-specs  # legacy `*_grid.yaml` を strategy YAML に移行
uv sync --group nautilus         # optional: real Nautilus runtime smoke 用依存を追加
uv run pytest tests/             # テスト
uv run ruff check src/           # リント
uv run pyright src/              # 型チェック
```

- Lab `evolve/optimize` の API/Web は `target_scope`（`entry_filter_only` / `exit_trigger_only` / `both`）を受け付ける（`entry_filter_only` は互換フラグとして維持）
- Lab `evolve/optimize` の frontend `allowed categories` は `all` / `fundamental only` を提供
- Lab frontend は `Run` / `History` タブを持ち、`/api/lab/jobs` で実行履歴を一覧し、選択したジョブの進捗・結果を再表示できる
- `Optimization` と candidate-producing な `Lab generate/evolve/optimize` は `engine_policy`（`fast_only` / `fast_then_verify`）と `verification_top_k`（`1..10`, default `5`）を受け付ける。`fast_then_verify` では `vectorbt` fast path 後に上位候補を `Nautilus` child backtest で直列 verification し、親 job は verification 完了まで `running` を維持し、API payload は `fast_candidates` / `verification` を返す
- Lab `evolve` は依存パラメータ制約付き mutation（`long>short` / `slow>fast` / `max>min`）を適用し、baseline（ベース戦略）より悪化した候補は guardrail で棄却して base 採用へフォールバックする
- Lab `evolve` は世代間で OHLCV/benchmark prefetch を再利用し、forecast revision が必要になった場合のみ再prefetch する
- Lab `optimize`（Optuna）は開始時に OHLCV/benchmark を1回プリフェッチして trial 間で再利用し、`pruning=true` 時は第1段階バックテストの暫定スコアで早期枝刈りを行う
- Lab `optimize`（Optuna）は依存パラメータ制約付きサンプリング（`long>short` / `slow>fast` / `max>min`）を適用し、`n_trials>=40` では `stage1(広域)+stage2(局所)` の2段階探索を行う
- Lab `optimize` は `GET /api/lab/optimize/recommendation` を提供し、戦略の探索次元数から `minimum/recommended/high_quality` trial 推奨値を返す。web の Optimize form は `target_scope` / `allowed_categories` 選択に追従して推奨値を再取得し、最低推奨未満を警告表示する
- Optimization HTML（`notebooks/templates/optimization_analysis.py`）は、各パラメータ組み合わせの `Trades`（closed trades件数）と Best detail の `Trade Count` を表示する
- `/api/optimize/jobs/{id}` は `best_score` / `total_combinations` に加えて `best_params` / `worst_score` / `worst_params` を返し、最適化ジョブ結果カードで best/worst 条件を比較表示できる
- `forward_eps_growth` / `peg_ratio` は FY実績EPSを分母に固定し、`period_type=FY` でも必要時のみ追加取得した四半期 FEPS 修正を forecast 側へ反映する
- Fundamental signal は `forecast_eps_above_recent_fy_actuals`（最新予想EPS > 直近FY X回の実績EPS最大値）をサポートし、`lookback_fy_count` で比較年数を指定できる。forecast revision 読み込み（screening/lab/optimization/backtest）を有効化する
- statements の share-adjusted per-share 指標（EPS/BPS/配当/forecast）は latest quarterly baseline shares を共通SoTとし、forecast revision 併用時も `base_df`/`revision_df` を同一 baseline で調整する。fundamentals/ranking も同じ基準解決を使い、比較軸の不整合を防ぐ
- Fundamental signal system は `cfo_margin` / `simple_fcf_margin`（売上高比マージン判定）をサポートし、`OperatingCashFlow` / `InvestingCashFlow` / `Sales` をデータ要件とする
- Fundamental signal は `cfo_to_net_profit_ratio`（営業CF/純利益）をサポートし、`consecutive_periods` 判定は比率値同値時でも開示更新（OperatingCashFlow/Profit）を起点に連続判定する
- Fundamentals は EPS に加えて `dividend_fy` / `forecast_dividend_fy` と `payout_ratio` / `forecast_payout_ratio`（実績/予想）を SoT とし、Charts の Fundamentals panel と Backtest Signal system（`forward_dividend_growth` / `dividend_per_share_growth` / `payout_ratio` / `forward_payout_ratio`）で同一指標を使う。配当性向は API 返却時に percent 単位へ正規化し、decimal スケール値（例: 0.283）を 28.3% として扱う
- fundamentals 最新値の forecast EPS は同一期末内で `DiscDate` が新しい開示を優先し、旧開示値の逆転表示を防ぐ
- Charts Fundamentals panel は `forecastEpsAboveRecentFyActuals`（最新予想EPS > 直近FY X回の実績EPS最大値）を latest metrics で返し、`forecastEpsLookbackFyCount` に応じた true/false を表示する（旧 `forecastEpsAboveAllHistoricalActuals` は互換フィールド）
- `/api/analytics/fundamental-ranking` は `market.duckdb`（`statements`/`stocks`/`stock_data`）を SoT とし、`metricKey` と `rankings.ratioHigh` / `rankings.ratioLow` を返す。現在の `metricKey` は `eps_forecast_to_actual`（最新の予想EPS / 最新の実績EPS）で、予想EPSは `revised(四半期) > adjusted FY forecast > raw FY forecast`、実績EPSは最新 FY EPS（share補正）を採用する。`forecastAboveRecentFyActuals=true` と `forecastLookbackFyCount` で「最新予想EPS > 直近FY X回の実績EPS最大値」条件を追加フィルタできる（旧 `forecastAboveAllActuals` も互換）。将来の比率指標追加は `metricKey` で識別する
- Financial analysis の計算ロジック SoT は `src/domains/*`（`analytics` / `fundamentals` / `strategy/indicators`）とし、`src/application/services/*` と `entrypoints` は I/O + orchestration に限定する
- Strategy group 再振り分けは `/api/strategies/{strategy_name}/move`（`target_category`: `production` / `experimental` / `legacy`）を SoT とし、web の `Backtest > Strategies` から実行する

主要技術: Python 3.12, vectorbt, pydantic, FastAPI, pandas, ruff, pyright, pytest

## ts (TypeScript / bun)
日本株式の解析を行うTypeScriptモノレポ。ランタイムは **bun** を使用。

| パッケージ | 役割 |
|---|---|
| `packages/web/` | React 19 + Vite フロントエンド |
| `packages/contracts/` | OpenAPI 生成型・API response 型・bt:sync |
| `packages/utils/` | logger/env/date/path などの共通ユーティリティ |
| `packages/api-clients/` | FastAPI クライアント（backtest/analytics/JQuants） |

```bash
bun run workspace:dev            # web 起動（FastAPI :3002 にプロキシ）
bun run workspace:dev:sync       # bt:sync + web:dev（sync失敗時はwarningで継続）
bun run workspace:test           # テスト
bun run quality:typecheck        # 型チェック
bun run quality:deps:audit       # 依存宣言 drift / 未使用依存監査
bun run quality:lint && bun run quality:check:fix  # リント（Biome）
bun run --filter @trading25/web e2e:smoke  # web E2E smoke（Playwright）
```
`main` ブランチでは `workspace:dev` を既定とし、`workspace:dev:sync` は OpenAPI 契約更新の確認が必要な場合のみ使う。

- `apps/ts` の依存は `bun run quality:deps:audit` を SoT に棚卸しし、未使用依存、script/import と manifest の不整合、root override と package version drift を検出する。`zustand` は完全撤去ではなく縮小方針とし、URL と相性の良い page selection state は TanStack Router search params を SoT にする
- Backtest UI は `Attribution` サブタブ内に `Run` / `History` を持ち、進捗取得は 2 秒ポーリング
- Backtest `Strategies` 画面の Strategy Editor は `production` / `experimental` の編集を許可し、`Visual / Advanced YAML / Preview` の hybrid editor、section sidebar、backend metadata-driven な `shared_config` / signal guidance、`dataset` / `benchmark_table` の reference select UI を提供する。`Rename` / `Delete` は `experimental` のみ許可
- Backtest `Default Config` editor は `default.execution` と `default.parameters.shared_config` を visual 編集でき、`dataset` / `benchmark_table` は Strategy Editor と同じ reference select card を使い、raw YAML は advanced fallback として維持する
- Strategy optimization の SoT は strategy YAML トップレベル `optimization` block とし、旧 `*_grid.yaml` sidecar / `/api/optimize/grid-configs*` は廃止する。`GET/POST draft/PUT/DELETE /api/strategies/{strategy}/optimization` を strategy-scoped API とし、CLI では `bt migrate-optimization-specs` で legacy sidecar を one-shot 移行する
- Backtest `Strategies > Optimize` は `Open Editor` ポップアップで Monaco + Signal Reference を表示し、saved spec が無ければ `Generate Draft from Strategy` を主導線にする。`Saved` / `Generated Draft` / `Drift` / `Ready to Run` を表示し、保存ブロックは YAML 構文エラーと構造エラー時のみ、warning は保存可能とする
- Backtest Runner の `Optimization` セクションは Grid 概要（params/combinations）に加えて `parameter_ranges` の具体値一覧を表示し、Optimization 完了カードでは Best/Worst Params と各 score を表示する
- Backtest `Optimization` / `Lab` form は `Fast only` / `Fast + Nautilus verify` と `Top K` を提供し、progress/history/result で fast stage と verification stage を分離表示する
- `screening`（web）は production 戦略を動的選択し、非同期ジョブ（2秒ポーリング）で実行する。`sortBy` 既定は `matchedDate`、`order` 既定は `desc`。`backtestMetric` は廃止。`markets` 未指定時は selected strategies（未選択なら eligible production strategies 全体）の dataset snapshot 実体銘柄 universe を Auto SoT とし、UI/API の scope label は dataset preset label を返す。explicit な request/URL/UI 指定が常に優先する
- `Screening / Ranking` の結果テーブルは大量件数時に virtualization を適用する
- frontend の desktop 作業幅はユーザーの常用レイアウト（おおむね 1180px 前後のコンテンツ幅）を優先し、hero/header/meta の縦占有を抑えて主テーブル・主チャートの可視領域を確保する
- Screening 画面は `Pre-Open Decidable / Requires In-Session Observation` の2タブ構成。Ranking 画面は `Daily Ranking / Fundamental Ranking` の2タブ構成で、Daily Ranking は `Individual Stocks / Indices` の2サブタブを持つ。Indices は backend `indexPerformance` を表示し、`lookbackDays` と `date` を共有 filter として使う。Fundamental Ranking は `Forecast High / Forecast Low / Actual High / Actual Low` の4サブタブで最新EPSランキングを表示する
- `/charts` `symbol`、`/portfolio` `tab|portfolioId|watchlistId`、`/indices` `code`、`/options-225` `date|putCall|contractMonth|strikeMin|strikeMax|sortBy|order`、`/screening` `tab + filter params`、`/ranking` `tab|dailyView + filter params`、`/backtest` `tab|strategy|resultJobId|dataset|labType` は Router search params を SoT にし、再訪/共有可能な UI 選択状態を URL で復元する
- Charts の sidebar 設定はカテゴリ別 Dialog（Chart Settings / Panel Layout / Fundamental Metrics / FY History Metrics / Overlay / Sub-Chart / Signal Overlay）で編集する。Fundamental 系パネル（Fundamentals / FY History / Margin Pressure / Factor Regression）は `fundamentalsPanelOrder` で表示順を保持・編集し、Fundamentals パネル内部の指標は `fundamentalsMetricOrder` / `fundamentalsMetricVisibility`、FY History パネル内部の指標は `fundamentalsHistoryMetricOrder` / `fundamentalsHistoryMetricVisibility` で順序・表示ON/OFFを保持する。Fundamentals パネル高さは表示中指標数に応じて動的に変化する
- Portfolio / Watchlist の銘柄追加入力はチャート検索と同等の銘柄サーチ（コード/銘柄名）を使う。追加送信 payload は `companyName` 必須（候補選択時は候補名、未選択時はコードをフォールバック）。Watchlist 追加の送信は 4 桁コードのみ許可する
- Fundamentals summary の予想EPS表示は `revisedForecastEps > adjustedForecastEps > forecastEps` の優先順位を SoT とする
- `N225 Options` ページは `/api/market/options/225` を SoT とし、DuckDB 同期済みの options chain を URL search params（`date|putCall|contractMonth|strikeMin|strikeMax|sortBy|order`）で探索する。live J-Quants proxy は診断用途に残すが、web 本導線では使わない
- `Indices` ページは synthetic category を `Benchmarks` として先頭表示し、`N225_UNDERPX` は true OHLC ではなく line chart + `UnderPx derived` 注記で扱う
- web `Market DB` ページは DuckDB SoT 同期 + DuckDB Snapshot 表示を提供し、`enforceBulkForStockData` を UI で切り替え可能とする。`Initial` mode では `resetBeforeSync` toggle と typed confirm dialog を出し、`market.duckdb` / `parquet/` を破壊的に reset してから再同期できる。`Warning Recovery` カードから `repair` sync を起動でき、listed-market fundamentals backfill など非 price warning の件数のみを表示する。legacy DB が検出された場合は reset enabled の `initial sync` を案内する。`/api/db/validate` の diagnostics は actionable warning と informational diagnostics を分離し、preferred-share issuer alias coverage、negative-cache sample codes、`options_225_data` の missing/stale/partial local coverage と UnderPx missing/conflict diagnostics を表示する。N225 options gap は `repair` ではなく Database Sync の `incremental` 導線で解消する。sync中は `GET /api/db/sync/jobs/{jobId}/fetch-details` をポーリングして stageごとの fetch strategy/execution（bulk/rest, endpoint, reason/fallback）を表示する。active sync job は `localStorage + /api/db/sync/jobs/active` で再読み込み/再訪時も復元し、running中は snapshot (`/api/db/stats` `/api/db/validate`) を短周期ポーリング + 進捗更新トリガで自動再取得する

主要技術: TypeScript, Bun, React 19, Vite, Tailwind CSS v4, Biome, OpenAPI generated types

## Issue管理

プロジェクトルートの `issues/`（オープン）、`issues/done/`（クローズ済み）で管理。
フォーマット: `{id}-{slug}.md`（例: `bt-016-test-coverage-70.md`）

## Skills ガバナンス

- プロジェクト正本のスキルは `/.codex/skills` に配置する
- `apps/ts/.claude` と `apps/bt/.claude` は廃止し、モノレポ正本の `/.codex/skills` のみを参照する
- 参照生成: `scripts/skills/refresh_skill_references.py`
- 監査: `scripts/skills/audit_skills.py --strict-legacy`（legacy ディレクトリ再混入の検知）

## CI

`.github/workflows/ci.yml` により全ブランチ push / PR で自動実行。
- push 前のローカル確認 SoT は `scripts/prepush-ci.sh`。`finish` を使う場合も script が存在すればこれを優先し、default は core CI mirror、`--full` は security audit + web e2e まで含める
- **skills**: audit（stale検知 / frontmatter検証 / legacy変更検知）
- **ts**: lint → 型生成 → build → typecheck → test + coverage
- **web e2e**: Playwright Chromium smoke（bt server :3002 を起動して実行）
- **bt**: lint → typecheck → test + coverage（ゲート70%）
- `.github/workflows/nautilus-smoke.yml` は `uv sync --locked --group nautilus` + `scripts/test-nautilus-smoke.sh` を使う separate CI で、default CI には混ぜない

## ロードマップ

現行インデックスは [`docs/unified-roadmap.md`](docs/unified-roadmap.md) を参照。  
Phase 1-4 の大規模リファクタリングは完了し、実行タスクは `issues/`（open）/`issues/done/`（closed）で管理する。  
旧統合ロードマップ本文は `docs/archive/unified-roadmap-2026-02-10.md` に archive 済み。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/d25037)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/d25037)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
