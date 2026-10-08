## book-gc-details

> このディレクトリは **ligarb** で管理された書籍プロジェクトです。書名などのメタデータは `book.yml` を参照。

# このプロジェクトについて

このディレクトリは **ligarb** で管理された書籍プロジェクトです。書名などのメタデータは `book.yml` を参照。

ligarb の仕様(コマンド、book.yml の書式、Markdown 拡張記法など)は `ligarb help` で確認できる。

# 編集ルール

- **下位見出しは単独で置かない**(lone subheading の禁止): ある (sub)section の直下に下位レベルの見出しを置くときは、同レベルの兄弟見出しを必ず 2 つ以上並べる。兄弟が 1 つしか作れないなら、見出しを立てずに本文や NOTE/TIP ボックスで書く。検査: `ruby tools/check_lone_headings.rb`(空出力なら OK)。

---
> Source: [kolanglab/book_gc_details](https://github.com/kolanglab/book_gc_details) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
