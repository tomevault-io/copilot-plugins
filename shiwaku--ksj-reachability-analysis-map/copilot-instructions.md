## ksj-reachability-analysis-map

> `input/origins.csv` に始点を記述してコミットすると GitHub Actions が自動的に到達圏分析を実行し、PMTiles + MapLibre GL JS ビューワーを GitHub Pages に公開するシステム。

# 到達圏分析マップ — Claude Code 向けプロジェクト仕様

## プロジェクト概要

`input/origins.csv` に始点を記述してコミットすると GitHub Actions が自動的に到達圏分析を実行し、PMTiles + MapLibre GL JS ビューワーを GitHub Pages に公開するシステム。

## ディレクトリ構成

```
.github/workflows/
  compute.yml             トリガー: input/origins.csv の push または手動

src/
  reachability_search.py  到達圏分析メイン（Dijkstra）
  to_geojson.py           parquet → GeoJSON（tippecanoe 入力用）
  ksj_to_network_csv.py   国土数値情報 GeoJSON → 道路リンク/ノード parquet
  make_access_links.py    L6 アクセスリンク生成

viewer/
  index.html              MapLibre GL JS + PMTiles ビューワー

data/
  prefecture.parquet      都道府県境界（--pref クリップ用）
  city.parquet            市区町村境界（--city クリップ用）

network/
  saitama_pref/           埼玉県サンプルネットワーク（リポジトリに同梱・gitignore 除外済み）
    KSJ_N13-24_saitama_pref_道路リンク.parquet   (61MB)
    KSJ_N13-24_saitama_pref_道路ノード.parquet   (18MB)
    KSJ_N13-24_saitama_pref_アクセスリンク_L6.parquet  (7.7MB)

input/
  origins.csv             始点リスト（lat,lon,name 列）← 編集してコミットで自動更新

output/                   gitignored（Actions が生成）
docs/                     gitignored（Actions が gh-pages ブランチにデプロイ）
```

## GitHub Actions ワークフロー（compute.yml）

```
push: input/origins.csv または workflow_dispatch
    ↓
pip install geopandas pyarrow scipy
    ↓
python3 src/reachability_search.py --orig-csv input/origins.csv
  → output/arrival_map_{name}.parquet（始点ごと）
  → output/origins_{names}.geojson
    ↓
sudo apt-get install tippecanoe
    ↓
python3 src/to_geojson.py
  → output/geojson/arrival_map_{name}.geojson（到達可能メッシュのみ・origin 列付き）
  → output/geojson/origins.geojson
    ↓
tippecanoe -z14 -Z6 -l reachability → docs/reachability.pmtiles
    ↓
cp viewer/index.html docs/index.html
cp output/geojson/origins.geojson docs/origins.geojson
    ↓
peaceiris/actions-gh-pages → gh-pages ブランチにデプロイ
```

## 主要スクリプト仕様

### reachability_search.py

- **パス定数**: `REPO_ROOT = Path(__file__).parent.parent`（`src/` の 1 つ上 = リポジトリルート）
- **デフォルトデータ**: `network/saitama_pref/`
- **出力**: `output/arrival_map_{name}.parquet`、`output/origins_{label}.geojson`
- **始点指定**: `--orig lat,lon,name`（複数可）または `--orig-csv path`

### to_geojson.py

- `output/arrival_map_*.parquet` → `output/geojson/arrival_map_{name}.geojson`
- `dist_min` が NaN（到達不能）のメッシュは除外
- `origin` 列（始点名）を追加（PMTiles でのフィルタリング用）
- `origins_*.geojson` を `output/geojson/origins.geojson` にコピー

### viewer/index.html

- MapLibre GL JS + pmtiles プロトコル
- PMTiles ソース: `pmtiles://reachability.pmtiles`（相対パス）
- `origins.geojson` を fetch して始点ドロップダウンを動的生成
- `dist_rank` で色分け（0〜9 の 10 分刻み）、フィルターで始点を切り替え
- 始点ポイントを白丸 + ラベルで表示

## 出力ファイル仕様

### arrival_map_{name}.parquet

| カラム | 内容 |
|---|---|
| `mesh_code` | L6 メッシュコード（11 桁） |
| `dist_min` | 最短到達時間（分）。到達不能は NaN |
| `dist_rank` | 10 分刻みランク（"0"〜"9"）。到達不能は "" |
| `geometry` | メッシュポリゴン（EPSG:4326） |

### GeoJSON（to_geojson.py 出力）

上記に `origin`（始点名）を追加。到達不能メッシュは含まない。

### PMTiles（reachability.pmtiles）

- レイヤー名: `reachability`
- ズーム: Z6〜Z14
- プロパティ: `mesh_code`, `dist_min`, `dist_rank`, `origin`

## 色定義（dist_rank）

| rank | 範囲 | 色 |
|---|---|---|
| 0 | 0〜10 分 | #ff0000 |
| 1 | 10〜20 分 | #ff4000 |
| 2 | 20〜30 分 | #ff8000 |
| 3 | 30〜40 分 | #ffc000 |
| 4 | 40〜50 分 | #ffff00 |
| 5 | 50〜60 分 | #c0ff00 |
| 6 | 60〜70 分 | #00cc00 |
| 7 | 70〜80 分 | #00cc80 |
| 8 | 80〜90 分 | #00cccc |
| 9 | 90 分超 | #440055 |

## GitHub Pages 設定

リポジトリの Settings → Pages → Source: **Deploy from a branch** → `gh-pages` / `/ (root)`

## 新しいエリアを追加する手順

1. 国土数値情報 GeoJSON を `input/N13-24_{mesh}/` に配置
2. `ksj_to_network_csv.py --meshes {meshes} --case {name} --pref {pref}` でネットワーク生成
3. `make_access_links.py --meshes {meshes} --case {name} --level 6 --pref {pref}` でアクセスリンク生成
4. `.gitignore` に `!network/{name}/` と `!network/{name}/**` を追記してコミット
5. `compute.yml` の `--links`/`--nodes`/`--access` をデフォルト値から変更するか、`reachability_search.py` 呼び出し時に明示指定

## 制約

- 一方通行未考慮（国土数値情報に一方通行フィールドなし）
- メッシュレベル L6（125m）固定
- vehicle モード（道路種別・幅員による速度テーブル）

---
> Source: [shiwaku/ksj-reachability-analysis-map](https://github.com/shiwaku/ksj-reachability-analysis-map) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
