## rippleisland-guide

> ゲーム「リップルアイランド」（カイルとキャルのレストラン）の攻略サイト。Astro + GitHub Pages（base: `/rippleisland-guide`）。

# リップルアイランド攻略ノート

ゲーム「リップルアイランド」（カイルとキャルのレストラン）の攻略サイト。Astro + GitHub Pages（base: `/rippleisland-guide`）。

## 構成

- `src/content/` — コンテンツ本体（Markdown + frontmatter）。スキーマは `src/content/config.ts`
  - `ingredients/` 素材、`equipment/` 置物（料理器具）、`recipes/` レシピ、`stages/` ステージ
- `src/pages/` — 一覧・詳細ページ（コレクションごとに index.astro と [slug].astro）
- 相互リンクは「参照を持つ側だけに書き、逆方向はビルド時に全件走査で逆引き」する方針
  - 器具→素材: `equipment.materials` / レシピ→器具: `recipes.equipment` / ステージ→素材・レシピ: `stages.ingredients` `stages.recipes`
  - 参照の比較は必ず `slug` で行う（`id` は存在しないので使わない）
- マルチプレイ専用ステージは `episode: multi` + `phases` 配列（1ページ内に全フェーズ縦積み、stickyジャンプナビ）
- 動作確認は `npm run build`。dev サーバーはポート4321（ユーザーが起動していることが多い）

## データ更新ワークフロー（重要）

ユーザーは `data/リップルアイランド攻略データ.xlsx` に記入して「このExcelをサイトに反映して」と依頼する。読み取りルール:

- シート: 使い方 / 素材 / 中間素材 / 置物（器具） / レシピ / ステージ / マルチステージ
- **クリーム色の行** = サイト登録済みデータ。内容が変わっていたら該当 `.md` を更新
- **黄色の行** = 記入例。「（記入例）」と書かれたものは取り込まない。ユーザーが実データに書き換えている場合は取り込む
- 複数値はカンマ区切りの slug。レシピの材料は `slug:数量`（例: `tomato:2個`）
- レシピの「カテゴリ」列は一覧ページのグループ分けに使う。表示順は 主食・軽食 / スープ / おかず・メイン / サラダ / デザート・スイーツ / ドリンク の6種。未設定・未知の値は一覧末尾の「その他」に集約される（グループ内は難易度順ソート）
- 攻略のポイントはセル内改行で1行1ポイント → frontmatter の配列に変換
- マルチステージは1行＝1フェーズ。同じ slug の行が1ステージ。ステージ名・アイコン・難易度・説明は最初の行のみ記入されている
- slug が空欄なら英小文字ケバブケースで命名する

### 未実装の宿題

- **中間素材（例: カットトマト）はサイト側が未実装。** Excelに中間素材データが来たら、`intermediates` 相当のコレクション・一覧・詳細ページ・加工チェーン表示（元素材→器具→中間素材）をあわせて実装すること。中間素材はレシピの材料や器具の対応素材としても参照される想定

## 進め方の約束

- 指揮・レビューは上位モデル、実装作業はSonnet等の下位モデルのサブエージェントに委任する
- コミットはユーザー自身が行う（Codexはコミットしない。依頼されたときのみ）
- コンテンツの文体・スタイルは既存ファイルに合わせる（日本語、CSS変数、ジャー型カード等）

---
> Source: [KakuMaru06/rippleisland-guide](https://github.com/KakuMaru06/rippleisland-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
