## claude-code-skill-esa-images

> esa記事から添付画像を一括ダウンロードする。esa の記事番号を指定すると、記事本文中の img.esa.io 画像URLを全て抽出し、ローカルにダウンロードする。第3引数で保存先ディレクトリを指定可能（省略時は .design/esa_{postNumber}/）。


esa記事の添付画像を一括ダウンロードする。

## 引数

- `$0` — esa チーム名（例: kaigionrails）
- `$1` — 記事番号（例: 826）
- `$2` — （省略可）保存先ディレクトリ。省略時は `.design/esa_$1/`

## 手順

1. esa MCP ツール `mcp__esa__esa_get_post` で teamName=$0, postNumber=$1 の記事を取得する
2. 記事本文（body_md）から `https://img.esa.io/uploads/` で始まる画像URLを全て抽出する
3. 保存先ディレクトリを決定する
   - `$2` が指定されている場合はそのディレクトリを使用する
   - 省略時はプロジェクトルートに `.design/esa_$1/` を作成する
4. `.gitignore` の処理（保存先が `.design/` 配下の場合のみ）
   - `.gitignore` に `.design/` が含まれていなければ追加する
   - 保存先が `.design/` 配下でない場合はスキップする
5. 各画像を curl でダウンロードする。ファイル名は連番 + 内容がわかる名前をつける（例: `01_top_page.png`）
   - Markdown中の alt テキストやコンテキストからファイル名を推測する
   - alt テキストがない場合は `01_image.png` のような連番にする
6. ダウンロード完了後、ファイル一覧とサイズを表示する
7. 最後に `Read` ツールで代表的な画像を1枚読んで、正常にダウンロードできたことを確認する

## 注意事項

- 画像URLが認証付きで403になった場合は、その旨をユーザーに伝える
- 同じディレクトリが既に存在する場合は、上書きするか確認する
- @4x 等のRetina画像の場合、その旨を伝える

---
> Source: [taea/claude-code-skill-esa-images](https://github.com/taea/claude-code-skill-esa-images) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
