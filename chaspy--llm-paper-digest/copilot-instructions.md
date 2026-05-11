## llm-paper-digest

> 1. `lists/reading-list.md` から status=pending の論文を1件取得

# llm-paper-digest Worker 指示書

## 処理フロー

1. `lists/reading-list.md` から status=pending の論文を1件取得
2. arxiv API で論文メタデータ取得: https://export.arxiv.org/api/query?id_list=<arxiv_id>
3. 論文 PDF または abstract から日本語詳細解説を生成 → `summaries/YYYY-MM-DD-<slug>.md`
4. Marp スライドを生成 → `slides/YYYY-MM-DD-<slug>.md`
5. `lists/reading-list.md` の status を pending→done に更新
6. 全ファイルを commit して push

## 出力ディレクトリ
- `papers/` — 論文メタデータ（title, authors, abstract, arxiv_id, date）
- `summaries/` — 日本語詳細解説
- `slides/` — Marp スライド

## arxiv API
- エンドポイント: https://export.arxiv.org/api/query?id_list=<arxiv_id>
- レスポンス: Atom XML
- title, summary (abstract), authors を取得

## 注意
- 1回の実行で1論文のみ処理
- status=done の論文はスキップ

---
> Source: [chaspy/llm-paper-digest](https://github.com/chaspy/llm-paper-digest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
