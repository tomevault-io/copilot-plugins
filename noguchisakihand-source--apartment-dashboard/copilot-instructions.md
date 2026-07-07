## apartment-dashboard

> Claude Code向けのプロジェクト開発ガイド。

# CLAUDE.md

Claude Code向けのプロジェクト開発ガイド。

## プロジェクト概要

東京12区+千葉3市の中古マンションお買い得物件ダッシュボード。
- 成約データ（不動産情報ライブラリAPI）と売出データ（SUUMO）を比較
- お買い得スコア = (相場価格 - 売出価格) / 相場価格 × 100

## データパイプライン

**依存順序があるため、以下の順序で実行すること：**

```
1. fetch_reinfolib.py  → transactions テーブル（成約データ）
2. scrape_suumo.py     → listings テーブル（売出データ）
3. geocode.py          → listings に latitude/longitude 追加
4. calc_market_price.py → market_prices テーブル（相場算出）
5. calc_deal_score.py  → listings に deal_score 追加
6. calc_comprehensive_score.py → listings に comprehensive_score 追加（総合スコア）
6. git push            → Streamlit Cloud に反映（1-2分）
```

**重要: 新地域追加時は 1-6 すべて実行必須**

## よく使うコマンド

### 全パイプライン実行（ワンライナー）

```bash
cd ~/apartment-dashboard && \
python3 scripts/fetch_reinfolib.py && \
python3 scripts/scrape_suumo.py && \
python3 scripts/geocode.py && \
python3 scripts/calc_market_price.py && \
python3 scripts/calc_deal_score.py &&
python3 scripts/calc_comprehensive_score.py
```

### DB確認コマンド

```bash
# 物件数（アクティブ）
sqlite3 data/apartment.db "SELECT COUNT(*) FROM listings WHERE status='active'"

# 地域別物件数
sqlite3 data/apartment.db "SELECT ward_name, COUNT(*) FROM listings WHERE status='active' GROUP BY ward_name ORDER BY COUNT(*) DESC"

# 成約データ数
sqlite3 data/apartment.db "SELECT ward, COUNT(*) FROM transactions GROUP BY ward"

# ジオコード済み件数
sqlite3 data/apartment.db "SELECT ward_name, COUNT(*), SUM(CASE WHEN latitude IS NOT NULL THEN 1 ELSE 0 END) as geocoded FROM listings WHERE status='active' GROUP BY ward_name"

# スコア算出件数
sqlite3 data/apartment.db "SELECT ward_name, COUNT(*), SUM(CASE WHEN deal_score IS NOT NULL THEN 1 ELSE 0 END) as scored FROM listings WHERE status='active' GROUP BY ward_name"

# 相場パターン数
sqlite3 data/apartment.db "SELECT ward, COUNT(*) FROM market_prices GROUP BY ward"
```

### ローカル起動

```bash
streamlit run app/app.py
# ブラウザで http://localhost:8501
```

### デプロイ

```bash
git add data/apartment.db data/geocode_cache.json
git commit -m "chore: DB更新"
git push origin main
# Streamlit Cloud: 1-2分で自動デプロイ
```

## 新地域追加チェックリスト

新しい区/市を追加する際は、**すべて**実行すること：

- [ ] `config/settings.yml` の `target_wards` に追加
- [ ] `scripts/fetch_reinfolib.py` の `WARD_CODES` に市区町村コード追加
- [ ] `scripts/scrape_suumo.py` の `AREA_CODES` に追加
- [ ] `python3 scripts/fetch_reinfolib.py` 実行
- [ ] `python3 scripts/scrape_suumo.py` 実行
- [ ] `python3 scripts/geocode.py` 実行
- [ ] `python3 scripts/calc_market_price.py` 実行
- [ ] `python3 scripts/calc_deal_score.py` 実行
- [ ] ローカルで `streamlit run app/app.py` 確認
- [ ] `git push origin main`
- [ ] Streamlit Cloud で反映確認

### 市区町村コード参照

- 東京都: 13101〜13123（特別区）
- 千葉県: 12203（市川市）、12207（松戸市）、12227（浦安市）

## テスト時の確認ポイント

### SUUMOスクレイピング

- [ ] 複数ページ取得できているか（1ページ目だけでないか）
- [ ] 総ページ数認識が正しいか（`page=1` で確認）
- [ ] フィルター後件数が妥当か（SUUMOサイトの検索結果と比較）

### 地域別確認クエリ

```bash
# 全指標を一覧表示
sqlite3 data/apartment.db "
SELECT
  ward_name,
  COUNT(*) as total,
  SUM(CASE WHEN latitude IS NOT NULL THEN 1 ELSE 0 END) as geocoded,
  SUM(CASE WHEN deal_score IS NOT NULL THEN 1 ELSE 0 END) as scored
FROM listings
WHERE status='active'
GROUP BY ward_name
ORDER BY ward_name
"
```

期待値：
- geocoded = total（全件ジオコード済み）
- scored > 0（スコア算出あり）

## 注意事項

### Streamlit Cloud
- push後1-2分で自動デプロイ
- 反映されない場合: https://share.streamlit.io/ でRebootボタン

### Git / DB
- `apartment.db` はバイナリファイル
- rebaseでコンフリクトすると解決困難
- `git push --force` は避ける
- リモートに先行変更がある場合は `git pull --rebase` してからpush

### APIレート制限
- SUUMO: 2秒wait/ページ（`scrape_suumo.py` 内で設定済み）
- 国土地理院API: 0.5秒wait/リクエスト
- 不動産情報ライブラリAPI: 特になし（ただし大量リクエスト注意）

### スコア未算出の原因
- 成約データ不足（min_sample_count=20件未満）
- 築年×面積ブラケットの組み合わせがない

## ディレクトリ構成

```
apartment-dashboard/
├── app/
│   └── app.py               # Streamlitアプリ（メイン）
├── scripts/
│   ├── fetch_reinfolib.py   # 成約データ取得（API）
│   ├── scrape_suumo.py      # 売出物件スクレイピング
│   ├── geocode.py           # 住所→緯度経度変換
│   ├── calc_market_price.py # 相場算出（区×築年×面積）
│   ├── calc_deal_score.py   # お買い得スコア計算
│   └── utils/
│       ├── config.py        # 設定読み込み
│       └── db.py            # DB接続ヘルパー
├── config/
│   └── settings.yml         # 対象地域設定
├── data/
│   ├── apartment.db         # SQLiteデータベース
│   └── geocode_cache.json   # ジオコードキャッシュ
├── .github/
│   └── workflows/
│       └── weekly_update.yml # 週次自動更新
├── requirements.txt
├── README.md
└── CLAUDE.md                # このファイル
```

## DBスキーマ（主要テーブル）

### listings（売出物件）
- suumo_id, property_name, ward_name, address
- asking_price, area, building_year, floor_plan
- station_name, minutes_to_station
- latitude, longitude（geocode.pyで追加）
- market_price, deal_score（calc_deal_score.pyで追加）

### transactions（成約データ）
- ward, trade_price, area, building_year, year, quarter

### market_prices（相場）
- ward, age_bracket, area_bracket, avg_price, sample_count

---
> Source: [noguchisakihand-source/apartment-dashboard](https://github.com/noguchisakihand-source/apartment-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
