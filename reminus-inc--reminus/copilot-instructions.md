## reminus

> note からブログ記事を取得してリポジトリに取り込む手順。`pnpm sync:blog` を叩くと

# Claude Code 向け運用メモ

## ブログ記事の同期

note からブログ記事を取得してリポジトリに取り込む手順。`pnpm sync:blog` を叩くと
この手順を実行するための Claude Code セッションが起動する想定。

### 手順

1. **同期スクリプトを実行する**
   - 単体: `pnpm sync:blog:run <noteKey> <slug>`
     - `noteKey` は note の記事 URL (`https://note.reminus.co.jp/n/xxxxx`) の `xxxxx` 部分
     - `slug` は `/blog/<slug>/` として配信される人間可読な英語スラッグ
       (例: `saas-ai-survival`)
   - 全件再同期: `pnpm sync:blog:run` (引数なし)
     - `src/app/blog/_articles/*.ts` から既存の `noteKey` ↔ `slug` を読み取り、
       RSS 上の記事を上書き更新する。新規記事は個別に単体コマンドで登録すること。
   - 画像は `public/blog/<slug>/` にダウンロードされ、本文中の src が
     ローカルパスに書き換えられる。

2. **末尾の note 内 CTA を剥がす (自己紹介テキストは残す)**
   - note の記事末尾には「Reminus（レミナス）について」のような自己紹介ブロック
     が付いてくることが多い。**自己紹介テキストそのものは残して OK**。
     サイト内記事とはいえ読み手にサービスを思い出してもらう文脈としては有用。
   - 剥がすのは **サイト遷移系の CTA**。記事末尾の `<ArticleCta />` (資料DL /
     お問い合わせボタン) と役割が被るので削除する。具体的には:
     - `<p><a href="https://reminus.co.jp/" ...>Reminus 公式サイト</a></p>`
     - `<p><a ...>Reminus 公式サイトで詳しく見る</a></p>`
     - `よければぜひ下記より…をご覧ください！` のような、直後のリンクを
       指している**オーファン文** (リンクを剥がした場合はこの文も一緒に削除)
     - `ご希望の方は<a href=".../contact">こちら</a>からどうぞ！` のような、
       お問い合わせ/ダウンロード先へ誘導する末尾の CTA 段落
     - 末尾の空の `<p><br></p>`
   - **どれが CTA かの判定は記事ごとに揺れる**。文言もレイアウトも記事によって
     違うので、機械的にマッチングさせず、その記事の contentHtml を読んで
     「これは自己紹介の本文か？ サイト遷移系 CTA か？」を都度判断すること。
     インラインで自然な文脈に溶けている参考リンクは残してよい。迷ったら
     ユーザーにどこまで剥がすか確認する。
   - 記事末尾に `<ArticleCta />` が配置されていることを確認する
     (`src/app/blog/[slug]/page.tsx`)。配置済みならボタンが variant に応じて
     `/download`・`/contact` または `/c/download`・`/c/contact` に飛ぶ。

3. **動作確認**
   - `pnpm tsc --noEmit` で型チェック
   - 必要に応じて dev server で `/blog/<slug>/` を目視確認

### 注意

- `_articles/<slug>.ts` は自動生成ファイルだが、上記の CTA 差し替えを施しiiyた
  **後に** コミットする。再同期すると `contentHtml` が上書きされるので、
  再同期のたびに 2. の手順をやり直す必要がある。
- `noteKey` は `_articles/<slug>.ts` 内に保持されているので、全件再同期時の
  マッピングに使われる。手で書き換えないこと。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Reminus-Inc) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
