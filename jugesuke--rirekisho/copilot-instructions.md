## rirekisho

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## リポジトリの概要

`comfortex`（日本語向け LaTeX 執筆環境テンプレート）をベースにした、履歴書用の TeX ドキュメントリポジトリ。ソースコードではなく **LaTeX 文書**が成果物であり、ビルドはローカルにインストールされた TeX ではなく **Docker イメージ `paperist/texlive-ja:latest`** 上で行う。Node/npm は textlint と npm-scripts のランナーとしてのみ使う。

構成:

- `rirekisho.sty` — 履歴書のレイアウト（厚労省「履歴書様式例（令和3年4月）」準拠）。
- `main.tex` — 唯一のビルド対象。記入用データのみを書く。`\shimei` などのコマンドで値を与え、`\rirekisho` で 2 ページを組版する。
- `blank.tex` — 空のひな形。`main.tex` を書き潰してしまったときはこれを写して戻す（ビルド対象ではない）。
- `example.tex` — 記入例（ビルド対象ではない。内容を `main.tex` に貼って使う）。

`blank.tex` / `example.tex` もひな形として `\documentclass` から `\end{document}` まで揃った完結した文書であり、`main.tex` と重複する。3 つのいずれかで書式コメントや使い方を直したら、他の 2 つにも同じ修正を反映すること。

レイアウト上の規約:

- 行の高さは**ストラット（`\rirekiRow`）か高さ指定セル（`\rirekiCell` / `\rirekiCellC`）のどちらか一方**で与える。両方使うと二重に加算される。表側の行ストラットは `\arraystretch` を 0 にして殺してある。
- 1 ページ目上部は **L 字**に組む。氏名欄・生年月日欄は電話欄の縦罫線より `\rirekiNameInset` だけ左で止め（`\rirekiFormLeftWidth`）、写真はその右のあきに `\rirekiPhotoOverlay` で**高さを持たないボックスとして重ねる**。写真は**表題行の上端から**始まり、左右は氏名欄の右端と版面右端の中央に置く。下端が現住所欄の上罫線の `\rirekiPhotoGap` 上にくるよう、**氏名行の高さ（`\riri@nameheight`）は表題行の実測高さから逆算する**（固定値を持たせないこと）。表題行の作成日も `\rirekiFormLeftWidth` に合わせる。
- 罫線は外枠 `\rirekiOuterRuleWidth`（0.8pt）と内側 `\rirekiInnerRuleWidth`（0.4pt）の2種類。外枠の横罫は `\rirekiHOut`、縦罫は列指定 `O` を使う（`!{\vrule ...}` を素のマクロで書くと array がプリアンブルを解釈できない）。
- 2 ページ目の先頭は `\vspace*` で 1 ページ目の表題行の分だけ下げ、最初の罫線の高さを揃えている。`\topskip` はここでは効かせたくないので差し引く。
- 枠と枠を縦につなぐときは必ず `\rirekiGlue`（`\par\nointerlineskip\noindent`）を挟む。`\noindent` だけだと直前の表と同じ行に並んでレイアウトが崩れる。枠どうしを離すときは代わりに `\rirekiSkip`（`\rirekiBlockSkip` のあきを入れる）を使う。
- `\multirow` は使わない。行の実際の高さを知らず（`\baselineskip` の段数と仮定する）、写真欄や電話欄の位置が狂う。複数行にまたがる欄は 1 つの高さ指定ボックスに組み、内部の区切り線は `\rirekiInnerRule` で引く（`\hrule` はセルの本文幅しか引かれず、両端に `\tabcolsep` のあきが残る）。
- 日付欄（作成日・生年月日）は `\sakuseibi{年}{月}{日}` / `\seinengappi{年}{月}{日}` と**算用数字**で与える。既定の表示は西暦（`2026年8月2日`）で、和暦などにしたいときは省略可能引数で表示だけ上書きする（`\sakuseibi[令和8年8月2日]{2026}{8}{2}`）。数値引数は計算に使うので、上書きしても和暦の数字を入れないこと。
- 作成日を空（`\sakuseibi{}{}{}`）にするとビルド日を西暦で入れる。ビルド日を和暦で出すなら `\sakuseibi[\wareki]{\the\year}{\the\month}{\the\day}`。
- 満年齢は**作成日を基準日として自動で計算する**（その年の誕生日前なら 1 引く）ので、**入力しない**。生年月日の年を空にすると生年月日・満年齢とも空欄の書式になる。計算は `\sakuseibi` と `\seinengappi` の記入順に依存しないよう `\AtBeginDocument` で行う（`\riri@setsakuseibi` → `\riri@setnenrei`）。表示用の値（`\riri@sakuseibi` / `\riri@seinengappi`）と計算用の数値（`\riri@kijunY` などの 6 つ）は別に保持している。
- 写真は `\shashin{ファイル名}` で入る。縦横比が `\rirekiPhotoRatioW`:`\rirekiPhotoRatioH`（3:4）でない画像は、短い方の辺を枠に合わせてから、はみ出す分を `trimclip` の `\clipbox` で中央基準に切る。未指定なら点線の貼付枠だけを出す。

学歴・職歴は `\gakureki` を書いた順に並び、1 ページ目の行数（`\rirekiRowsGakurekiA`＝15）を超えた分は自動で 2 ページ目（`\rirekiRowsGakurekiB`＝7）へ送られる。免許・資格（`\menkyo`）は 2 ページ目の `\rirekiRowsMenkyo`＝6 行のみで、次ページへの送りはない。**どちらも枠数を超えた項目は警告なく捨てられる**ので、項目を増やしたら PDF に全部出ているかを目で確かめる。行数や欄の高さを変えたときは、2 ページに収まるか（`Output written on main.pdf (2 pages, ...)`）も必ず確認する。

## コマンド

```bash
npm run init    # Docker イメージの pull と ./out ディレクトリ作成（初回のみ）
npm run build   # Docker 内で latexmk main.tex を実行 → out/main.pdf
npm run clean   # 中間ファイルの削除（latexmk -c）
npm run lint    # 全 *.tex（main / blank / example）に textlint を実行
```

- ビルドは必ず上記の npm-scripts 経由で行う（`docker run` を直接叩かない）。TTY のない環境（CI・非対話シェル）でも動くよう、`build` / `clean` からは `-it` を外してある。
- 単一ファイルだけを lint する場合: `npx textlint main.tex`
- テストは存在しない。品質チェックは textlint とビルドの成否のみ。

## ビルド構成

- `.latexmkrc` がビルド挙動の中心。エンジンは **LuaLaTeX**（`$pdf_mode = 4`）、文献は biber、索引は mendex。出力先は `./out`（`.gitignore` 済み）。
- `main.tex` は `jlreq` クラス（日本語組版）を使用。日本語フォントの都合上、LuaLaTeX 以外に切り替えないこと。
- `.vscode/settings.json` で LaTeX Workshop も同じ Docker イメージ経由でビルドするよう設定済み（`editor.formatOnSave` は有効だが LaTeX の整形は `none`）。

## textlint の方針

文体・句読点などの表記ルールは `.textlintrc.json` が唯一の情報源。文書を書く前にその内容を読み，設定に従うこと。

## CI

- `.github/workflows/review.yaml` — PR に対して textlint を実行し、reviewdog で PR レビューコメントを付ける。
- `.github/workflows/tex-preview.yaml` — 現在 `workflow_dispatch` のみ（自動実行はコメントアウト）。ビルドした PDF を Dropbox にアップロードし、共有リンクを PR にコメントする。Dropbox 用の secrets（`DROPBOX_APP_KEY` / `DROPBOX_APP_SECRET` / `DROPBOX_REFRESH_TOKEN`）が必要。

---
> Source: [jugesuke/rirekisho](https://github.com/jugesuke/rirekisho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
