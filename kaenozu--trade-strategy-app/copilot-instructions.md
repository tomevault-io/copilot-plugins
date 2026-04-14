## trade-strategy-app

> Streamlitベースの日本株トレード戦略アプリ。95銘柄のテクニカル分析・バックテスト・ウォークフォワード検証を行う。

# 日本株トレード戦略アプリ (trade-strategy-app)

Streamlitベースの日本株トレード戦略アプリ。95銘柄のテクニカル分析・バックテスト・ウォークフォワード検証を行う。

## プロジェクト構造

```
├── app.py                    # Streamlitダッシュボード（メインUI）
├── components/               # タブ別コンポーネント
│   ├── sidebar.py            # サイドバー設定UI
│   ├── dashboard.py          # ダッシュボードタブ
│   ├── dashboard_portfolio.py # ポートフォリオ表示
│   ├── charts.py             # チャート表示
│   ├── signals.py            # 戦略シグナル表示
│   ├── backtest_tab.py       # バックテストタブ
│   ├── risk_tab.py           # リスク管理タブ
│   ├── performance_tab.py    # パフォーマンスタブ
│   ├── constants.py          # UI定数（シグナル絵文字等）
│   └── __init__.py           # コンポーネントエクスポート
├── strategies/               # 個別戦略モジュール
│   ├── bb_reversal.py        # BB反転戦略
│   ├── trend_follow.py       # トレンドフォロー戦略
│   ├── breakout.py           # ブレイクアウト戦略
│   ├── mean_reversion.py     # 平均回帰戦略
│   ├── composite.py          # 総合判定（composite_score ラッパー）
│   ├── sector_rotation.py    # セクターローテーション戦略
│   └── __init__.py           # 戦略エクスポート
├── strategy_engine.py        # 戦略レジストリ・実行・シグナル取得
├── signal_utils.py           # シグナルユーティリティ（LatestSignal等）
├── backtest.py               # バックテスト（ATR損切り・利確・トレーリングストップ）
├── exit_logic.py             # 退出ロジック（損切り・利確・保有日数）
├── risk_manager.py           # ポジションサイジング（2%ルール・かぶミニ対応）
├── indicators.py             # テクニカル指標（SMA/RSI/MACD/BB/ATR/StochRSI/ADX）
├── config/                   # 全パラメータ・95銘柄リスト（パッケージ化）
│   ├── __init__.py           # 定数のエクスポート
│   ├── stocks.py             # 銘柄リスト・ユニバース・プレイブック
│   ├── trade_params.py       # トレードパラメータ
│   ├── indicators.py         # 指標パラメータ
│   └── data.py               # データ取得設定
├── data_fetcher.py           # yfinance経由OHLCV取得（parquetキャッシュ）
├── stock_screener.py         # 予算フィルタリング・銘柄スコアリング
├── portfolio_db.py           # SQLiteポートフォリオ管理
├── walk_forward.py           # ウォークフォワードテスト
├── market_regime.py          # マーケットレジーム検出
├── strategy_rotation.py      # 動的戦略選択
├── composite_scoring.py      # 総合スコアリング（Zスコア正規化）
├── portfolio_backtest.py     # ポートフォリオレベルバックテスト
├── portfolio_helpers.py      # ポートフォリオ内部ヘルパー
├── performance_reporting.py  # パフォーマンスレポート・実行ゲート
├── trade_utils.py            # トレードユーティリティ
├── daily_signal.py           # デイリーシグナル生成
├── operations.py             # 自動運用オペレーション
├── cli_report.py             # CLIレポート
├── tests/                    # テスト（805テスト・全パス）
│   ├── conftest.py           # テスト用フィクスチャ
│   ├── helpers.py            # テストヘルパー
│   ├── test_strategy_engine.py
│   ├── test_risk_manager.py
│   ├── test_stock_screener.py
│   ├── test_indicators.py
│   ├── test_data_fetcher.py
│   ├── test_walk_forward.py
│   ├── test_market_regime.py
│   ├── test_strategy_rotation.py
│   ├── test_backtest.py
│   ├── test_composite_scoring.py
│   ├── test_portfolio_backtest.py
│   ├── test_performance_reporting.py
│   ├── test_e2e_workflow.py
│   ├── test_benchmarks.py
│   ├── test_daily_signal.py
│   ├── test_performance_tab.py
│   ├── test_public_api.py
│   ├── test_sector_rotation.py
│   └── ...（他テストファイル）
├── scripts/                  # 分析・評価スクリプト（検証用）
│   ├── optimize_*.py         # 最適化スクリプト
│   ├── eval_*.py             # 評価スクリプト
│   ├── test_wf_*.py          # WF検証スクリプト
│   └── analyze_*.py          # 分析スクリプト
└── reports/                  # 最適化レポート・結果CSV
    ├── *_REPORT.md           # 分析レポート
    └── *_results.csv         # 結果データ
```

## 開発ルール

### 必須
- **mainへの直接プッシュ禁止**。全てPR経由でマージ。
- **変更前に必ずテスト実行**: `python -m pytest tests/ -v` → 全テスト通過必須
- **ウォークフォワード検証が基準**: 戦略変更は必ずOOS PnLで評価。IS成績だけで判断しない。
- 変更はブランチ→PR→レビュー→マージ

### コードスタイル
- コメントは不要（コードは自明に書く）
- 日本語のdocstringは既存パターンに従う
- 型ヒントは既存コードに合わせる
- importは `config/` の定数を活用（マジックナンバー禁止）

### テスト
- テストフレームワーク: pytest（850テスト・全パス）
- カバレッジ: 100%
- 全テストファイルは `tests/` に配置
- テスト実行: `python -m pytest tests/ -v`
- 高速化: `python -m pytest tests/ -q -n auto --dist=loadfile`

### ウォークフォワード検証
- `walk_forward.py` は `window_type` パラメータで `"expanding"`（デフォルト）と `"sliding"` をサポート
- ポートフォリオ制約検証: `test_wf_portfolio_constraints.py`
- ベースライン: BB反転 OOS PnL +49,915円（95銘柄・5年・5分割・sliding）
- 戦略ローテーション: OOS PnL +71,824円（+43.9%改善）
- 検証スクリプト: `scripts/verify_sliding_window_wf.py`（expanding vs sliding 比較）
- 改善案は個別にテストし、ベースラインを上回るもののみ採用

### レジーム連動戦略選択
- `strategy_rotation.py` でマーケットレジームに基づく戦略選択
- `portfolio_backtest.simulate_portfolio(strategy_key="strategy_rotation")` で銘柄ごとに最適戦略を自動選択
- レジーム判定: `market_regime.detect_regime()`（指数: ^N225、yfinance使用）
- バックテストタブ: `_get_rotation_result()` でローテーション結果を自動追加表示
- レジーム別推奨戦略:
  - `bull`: weekly_breakout, momentum, trend_follow, breakout, composite
  - `bear`: bb_reversal, weekly_mean_reversion, vix_timing
  - `range`: bb_reversal, composite, weekly_mean_reversion
  - `volatile`: composite, weekly_mean_reversion, mean_reversion, vix_timing
- スコアブースト: `REGIME_BOOST_FACTOR = 1.2`（推奨戦略のスコアを1.2倍）

## 戦略の現状

### 個別戦略 OOS成績（95銘柄・sliding・5y・5split）

| 戦略 | 個別OOS成績（95銘柄） | トレード数 | プラス銘柄率 | 状態 |
|---|---|---|---|---|
| BB反転（デフォルト） | +49,915円 | - | - | 推奨（ベースライン） |
| 平均回帰 | **+42,817円**（30銘柄V2） | - | - | 改善済み（V2） |
| ブレイクアウト | **+564,440円** | 662 | 63% | 改善済み |
| 総合判定 | **+399,075円** | 343 | 66% | 改善済み |
| トレンドフォロー | **+302,017円** | 450 | 65% | 改善済み |
| 週足ブレイクアウト | **+113,873円**（WF検証） | 982 | 62%（805/805） | 検証済み |
| セクターローテーション | **+138,915円**（95銘柄V2） | 1,224 | 64%（805/805） | 改善済み（V2・BB反転+178.3%） |
| **VIXタイミングV3** | **+361,518円**（95銘柄） | 1,636 | **51.6%** | 71.6% | ✅ 推奨（最高成績） |
| **モメンタム** | **+241,609円**（93銘柄・3分割） | 862 | 47.0% | 61.1% | ✅ 推奨 |
| **ギャップアップV6** | 検証中（トレード数59） | 59 | 42.4% | 17.9% | ⚠️ 日本株では機能しにくい |

### 戦略比較（WF OOS・95銘柄）

| 戦略 | OOS PnL | トレード数 | 平均勝率 | プラス銘柄 |
|---|---|---|---|---|
| **VIXタイミングV3** | **+361,518円** | 1,636 | 51.6% | 68/95 |
| **週足ブレイクアウト** | **+113,873円** | 982 | 39.2% | 59/95 |
| **モメンタム** | **+241,609円** | 862 | 47.0% | 58/95 |
| 週足トレンドフォロー | +38,157円 | 535 | 31.9% | 54/95 |
| 週足平均回帰 | +37,838円 | 322 | 27.8% | 51/95 |
| 日足ブレイクアウト | +57,742円 | 738 | 36.2% | 52/95 |
| BB反転 | +30,478円 | 422 | 33.3% | 54/95 |

**VIXタイミングV3は週足ブレイクアウトを+217.4%上回る最高成績（個別WF）**

### ポートフォリオバックテスト比較（95銘柄）

| 戦略 | PnL | トレード数 | 勝率 | PF | 最大DD |
|---|---|---|---|---|---|
| **VIXタイミングV3** | **+83,043円** | 2,164 | 51.1% | - | 3.71% |
| **戦略ローテーション** | **+54,326円** | 1,352 | 49.8% | 1.36 | 7.81% |
| BB反転 | +58,767円 | 1,140 | 56.1% | - | 3.91% |
| **モメンタムV2** | **+38,583円** | 1,633 | 43.8% | - | **7.68%** |
| **ギャップアップV8.5** | +146円 | 324 | 41.4% | - | 1.05% |
| 週足ブレイクアウト | +21,723円 | 1,505 | 40.1% | 1.13 | 13.69% |

**考察**: VIXタイミングV3が95銘柄ポートフォリオで最高のPnL。
モメンタムV2はスコア閾値引き上げによりPnL・勝率向上、DD削減に成功。

### ポートフォリオパラメータ最適化（95銘柄）

| max_positions | stop_loss | PnL | トレード数 | 勝率 | PF | 最大DD | リスク調整済みPnL |
|---|---|---|---|---|---|---|---|
| **5（最適）** | **2.0（最適）** | **54,880円** | 432 | 38.9% | 1.38 | **3.95%** | 11,087 |
| 25 | 2.0 | 75,838円 | 1782 | 38.4% | 1.30 | 8.00% | 8,426 |
| 25 | 3.0（デフォルト） | 46,514円 | 1665 | 41.1% | 1.27 | 7.27% | 5,624 |
| 10 | 2.5 | 32,773円 | 781 | 40.7% | 1.34 | 4.13% | 6,388 |
| 15 | 3.0 | 33,637円 | 1095 | 41.8% | 1.29 | 4.56% | 6,050 |

**考察**: max_positions=5, stop_loss=2.0がDD最小（3.95%）。PnLは25/2.0が最高（75,838円）だがDD 8.00%。リスク調整済みPnLでは5/2.0が最適。

### 戦略別パラメータ詳細

#### BB反転（推奨）
- 適応型BB（高ボラ時はperiod=10、通常は20）
- 買い: `near_lower & rsi < 40 + buffer & vol_ratio > 1.0 & ~crash_continuation`
- 売り: `near_upper & rsi > 70 - buffer & vol_ratio > 1.0`
- StochRSI < 25 かつ ADX < 25 で BUY、否则 BUY_WEAK

#### トレンドフォロー（改善済み）
- 買い: `sma_cross_signal == 2`（ゴールデンクロスイベント） & `vol_ratio > 0.8` & `rsi < 70`
- 売り: `sma_cross_signal == -2`（デッドクロスイベント）
- `TREND_VOL_RATIO_BUY = 0.8`（BB反転より緩和）

#### ブレイクアウト（改善済み）
- `BREAKOUT_PERIOD = 10`（10日高値突破）
- `BREAKOUT_VOLUME_MULT = 1.0`（出来高確認）
- SMAフィルター除去（20日高値突破が十分なシグナル）
- 買い: `close > high_breakout & vol_ratio > 1.0`

#### 総合判定（改善済み）
- 6指標（RSI/MACD/SMA/BB/Vol/ADX）の加重スコア
- BB成分反転: 下部バンド付近=高スコア（BB反転と同じ方向性）
- MACD/SMA: Zスコア正規化（200日標準偏差）でスケール圧縮解消
- ADX: `direction * (adx/50)` でトレンド強度重み付け
- ウェイト: RSI=15%, MACD=20%, SMA=20%, BB=20%, Vol=12.5%, ADX=12.5%
- 閾値: BUY≥25, BUY_WEAK≥10, SELL≤-20, SELL_WEAK≤-5

## 主要設定値 (config/)

### テクニカル指標
| 定数 | 値 | 説明 |
|---|---|---|
| `SMA_SHORT` | 5 | 短期SMA |
| `SMA_LONG` | 25 | 長期SMA |
| `SMA_TREND` | 75 | トレンド判断SMA |
| `RSI_PERIOD` | 14 | RSI期間 |
| `RSI_OVERSOLD` | 30 | RSI売られすぎ |
| `RSI_OVERBOUGHT` | 70 | RSI買われすぎ |
| `BB_PERIOD` | 20 | ボリンジャーバンド期間 |
| `BB_STD` | 2.0 | 標準偏差倍率 |
| `ATR_PERIOD` | 14 | ATR期間 |
| `ADX_THRESHOLD` | 25 | ADX閾値 |
| `STOCH_RSI_BUY_THRESHOLD` | 25 | StochRSI買い閾値 |

### 損切り・利確
| 定数 | 値 | 説明 |
|---|---|---|
| `STOP_LOSS_ATR_MULT` | 3.0 | 損切り（ATR×3.0） |
| `TAKE_PROFIT_ATR_MULT` | None | 利確無効（シグナル売りで退出） |
| `TRAILING_STOP_ATR_MULT` | None | トレーリングストップ無効 |
| `MAX_HOLDING_DAYS` | 22 | 最大保有日数 |

### リスク管理
| 定数 | 値 | 説明 |
|---|---|---|
| `RISK_PER_TRADE` | 0.02 | 資金の2% |
| `CONSECUTIVE_LOSSES_THRESHOLD` | 5 | 連続損失閾値 |
| `RISK_REDUCTION_FACTOR` | 0.5 | リスク削減係数 |
| `MAX_POSITIONS` | 25 | 最大ポジション数 |
| `MAX_CAPITAL_USAGE_PCT` | 0.95 | 最大資本使用率 |

### ポートフォリオ制約
| 定数 | 値 | 説明 |
|---|---|---|
| `PORTFOLIO_MAX_NEW_POSITIONS_PER_DAY` | 10 | 同日新規上限 |
| `PORTFOLIO_MAX_POSITIONS_PER_SECTOR` | 3 | セクター同時上限 |

### 実行ゲート
| 定数 | 値 | 説明 |
|---|---|---|
| `EXECUTION_PAUSE_WINDOW_DAYS` | 14 | 一時停止判定窓 |
| `EXECUTION_PAUSE_MAX_PROFIT_FACTOR` | 0.8 | PF閾値 |
| `EXECUTION_PAUSE_MIN_CLOSED_TRADES` | 10 | 最小決済数 |
| `EXECUTION_RESUME_WINDOW_DAYS` | 90 | 再開判定窓 |
| `EXECUTION_RESUME_MIN_PROFIT_FACTOR` | 1.0 | 再開PF閾値 |
| `EXECUTION_RESUME_MIN_CLOSED_TRADES` | 5 | 再開最小決済数 |

### 総合スコア閾値（Zスコア正規化後）
| 定数 | 値 |
|---|---|
| `COMPOSITE_BUY_THRESHOLD` | 25 |
| `COMPOSITE_BUY_WEAK_THRESHOLD` | 10 |
| `COMPOSITE_SELL_THRESHOLD` | -20 |
| `COMPOSITE_SELL_WEAK_THRESHOLD` | -5 |

#### 平均回帰（V2: グリッドサーチ最適化済み）
- 買い: `z_score < -1.2 AND (rsi < 45 OR vol_ratio > 0.8) AND adx < 30`
- 売り: `z_score > 1.0 OR rsi > 60`
- 30銘柄WF OOS: +42,817円（BB反転+25,644円を+67%改善）
- 最適化パラメータ: Z_SELL=1.0, RSI_BUY=45, ADX_MAX=30

#### 平均回帰（V2: グリッドサーチ最適化済み）
- 買い: `z_score < -1.2 AND (rsi < 45 OR vol_ratio > 0.8) AND adx < 30`
- 売り: `z_score > 1.0 OR rsi > 60`
- 30銘柄WF OOS: +42,817円（BB反転+25,644円を+67%改善）
- 最適化パラメータ: Z_SELL=1.0, RSI_BUY=45, ADX_MAX=30

#### セクターローテーション（V2: 95銘柄WF検証済み）
- モメンタムスコア: `mom_short * 0.6 + mom_long * 0.4`
- 買い: `momentum_score > 0 & trend_up & vol_ok & rsi_ok`
- 売り: `momentum_score < -0.02 OR rsi > 75 OR ~trend_up`
- Strong Buy: `momentum_score > rolling(20).quantile(0.7) & vol_ratio > 1.2`
- Weak Buy: `momentum_score <= 0.01 & vol_ratio <= 1.0`
- Weak Sell: `momentum_score > -0.05 & rsi < 70`
- **95銘柄WF OOS: +138,915円**（BB反転+49,915円を**+178.3%改善**）
- トレード数: 1,224、プラス銘柄率: 64%（805/805）

##### 週足ブレイクアウトパラメータ詳細
| 定数 | 値 | 説明 |
|---|---|---|
| `WEEKLY_BREAKOUT_PERIOD` | 8 | 週足高安値ブレイクアウト期間 |
| `WEEKLY_BREAKOUT_VOLUME_MULT` | 0.5 | 出来高比率倍率 |
| `VOL_FILTER_ATR_MULT` | 1.5 | ボラティリティフィルタ（ATR/20日平均） |

- 買い: `close > high_breakout(8週間) & vol_ratio > 0.5 & atr <= atr_ma20 * 1.5`
- 売り: `close < low_breakout(8週間) & vol_ratio > 0.5`
- **ボラティリティフィルタ**: ATRが20日平均の1.5倍以上時はエントリー抑制（高ボラでの損失回避）
- WF OOS: +113,873円（BB反転+228.1%）
- 期待効果: ポートフォリオDD 13.69% → 改善

##### 週足トレンドフォローパラメータ詳細
| 定数 | 値 | 説明 |
|---|---|---|
| `SMA_SHORT` | 5 | 短期SMA（週足） |
| `SMA_LONG` | 25 | 長期SMA（週足） |
| `TREND_VOL_RATIO_BUY` | 0.8 | 出来高比率倍率 |

- 買い: `sma_cross_signal == 2 & vol_ratio > 0.8 & rsi < 70`
- 売り: `sma_cross_signal == -2`
- WF OOS: +38,157円（BB反転-23.5%）

##### 週足平均回帰パラメータ詳細
| 定数 | 値 | 説明 |
|---|---|---|
| `WEEKLY_MR_LOOKBACK` | 50 | Zスコア算出期間（週足） |
| `WEEKLY_MR_Z_BUY` | -1.5 | 買いZスコア閾値 |
| `WEEKLY_MR_Z_SELL` | 0.0 | 売りZスコア閾値 |
| `WEEKLY_MR_RSI_BUY` | 40 | 買いRSI閾値 |
| `WEEKLY_MR_RSI_SELL` | 60 | 売りRSI閾値 |
| `WEEKLY_MR_ADX_MAX` | 25 | ADX上限 |

- 買い: `z_score < -1.5 & rsi < 40 & adx < 25`
- 売り: `z_score > 0.0 | rsi > 60`
- WF OOS: +37,838円（BB反転-24.1%）

##### セクターローテーションパラメータ詳細
| 定数 | 値 | 説明 |
|---|---|---|
| `SECTOR_MOM_SHORT_WEIGHT` | 0.6 | 短期モメンタム重み |
| `SECTOR_MOM_LONG_WEIGHT` | 0.4 | 長期モメンタム重み |
| `SECTOR_VOL_RATIO_MULT` | 0.8 | 出来高比率倍率 |
| `SECTOR_SELL_MOM_THRESHOLD` | -0.02 | 売りモメンタム閾値 |
| `SECTOR_SELL_RSI_THRESHOLD` | 75 | 売りRSI閾値 |
| `SECTOR_STRONG_BUY_WINDOW` | 20 | Strong Buy判定窓 |
| `SECTOR_STRONG_BUY_QUANTILE` | 0.7 | Strong Buy分位数 |
| `SECTOR_STRONG_BUY_VOL` | 1.2 | Strong Buy出来高比率 |
| `SECTOR_WEAK_BUY_MOM` | 0.01 | Weak Buyモメンタム |
| `SECTOR_WEAK_BUY_VOL` | 1.0 | Weak Buy出来高比率 |
| `SECTOR_WEAK_SELL_RSI` | 70 | Weak Sell RSI閾値 |
| `SECTOR_WEAK_SELL_MOM` | -0.05 | Weak Sellモメンタム |

##### モメンタムパラメータ詳細（新戦略）
| 定数 | 値 | 説明 |
|---|---|---|
| `MOMENTUM_LOOKBACK` | 20 | リターン計算期間（日） |
| `MOMENTUM_RSI_BUY_MAX` | 70 | 買いRSI上限 |
| `MOMENTUM_RSI_SELL_MIN` | 75 | 売りRSI下限 |
| `MOMENTUM_VOL_BUY_MIN` | 0.8 | 買い出来高比率下限 |
| `MOMENTUM_SCORE_SELL` | -5 | 売りスコア閾値 |

##### ギャップアップパラメータ詳細（新戦略）
| 定数 | 値 | 説明 |
|---|---|---|
| `GAP_UP_MIN_PCT` | 0.5 | 最小ギャップ率（%） |
| `GAP_UP_VOL_BUY_MIN` | 1.0 | 買い出来高比率下限 |
| `GAP_UP_RSI_BUY_MAX` | 65 | 買いRSI上限 |
| `GAP_UP_RSI_SELL_MIN` | 75 | 売りRSI下限 |

##### VIXタイミングパラメータ詳細（V3: 95銘柄WF検証済み）
| 定数 | 値 | 説明 |
|---|---|---|
| `VIX_LOW_THRESHOLD` | 25 | 冷静局面（VIX下限） |
| `VIX_HIGH_THRESHOLD` | 30 | 恐怖局面（VIX上限） |
| `VIX_DROP_THRESHOLD` | -5 | VIX急落（%） |
| `VIX_SPIKE_THRESHOLD` | 10 | VIX急騰（%） |
| `VIX_RSI_BUY_MAX` | 65 | 買いRSI上限 |
| `VIX_RSI_SELL_MIN` | 70 | 売りRSI下限 |

- 買い: `VIX < 25 or VIX急落5% & RSI < 65 & close > SMA20`
- 売り: `VIX > 30 or VIX急騰10% or RSI > 70`
- **95銘柄WF OOS: +361,518円**（最高成績）
- **ポートフォリオ: +83,043円**（95銘柄・勝率51.1%・DD 3.71%）

##### 平均回帰パラメータ詳細
| 定数 | 値 | 説明 |
|---|---|---|
| `MEAN_REV_LOOKBACK` | 30 | Zスコア算出期間（20→30） |
| `MEAN_REV_Z_BUY` | -1.2 | 買いZスコア閾値 |
| `MEAN_REV_Z_SELL` | 1.0 | 売りZスコア閾値（0.8→1.0） |
| `MEAN_REV_RSI_BUY` | 45 | 買いRSI閾値（35→45） |
| `MEAN_REV_RSI_SELL` | 60 | 売りRSI閾値（55→60） |
| `MEAN_REV_VOL_BUY` | 0.8 | 買い出来高閾値 |
| `MEAN_REV_ADX_MAX` | 30 | ADX上限（新設） |

### その他
| 定数 | 値 | 説明 |
|---|---|---|
| `COMMISSION_RATE` | 0.0 | 楽天証券ゼロコース |
| `SLIPPAGE_PCT` | 0.001 | スリッページ0.1% |
| `VOLUME_RATIO_BUY` | 1.0 | 出来高比率閾値 |
| `TREND_VOL_RATIO_BUY` | 0.8 | トレンドフォロー出来高閾値 |
| `REGIME_BOOST_FACTOR` | 1.2 | レジーム推奨スコアブースト |
| `REGIME_BEAR_HOLDING_MULT` | 0.5 | ベア相場保有期間倍率 |
| `REGIME_VOLATILE_HOLDING_MULT` | 0.75 | 変動相場保有期間倍率 |

### 自動運用設定（Windowsタスクスケジューラ対応）

`scripts/run_daily_ops.bat` をタスクスケジューラに登録することで、以下の処理を毎日自動化できます。

1.  `git pull` によるコードの最新化
2.  `daily_signal.py` によるシグナルスキャン（VIX V3）
3.  結果の標準出力（Slack通知と組み合わせる場合は `--slack` オプションを追加）

**使用例:**
```bash
# バッチファイルの実行
scripts\run_daily_ops.bat

# Slack通知を行う場合
python daily_signal.py --strategy vix_timing --slack
```

## データ

- キャッシュ: `.cache/` にparquet形式で保存（日付ごと）
- 取得期間: 5年（`5y`）
- 95銘柄: 東京エレクトロン、ソフトバンクG、みずほFG等（config/stocks.pyのSTOCKS参照）
- 指数: ^N225（日経平均、yfinance使用）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kaenozu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
