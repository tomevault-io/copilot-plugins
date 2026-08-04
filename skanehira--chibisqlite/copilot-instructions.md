## chibisqlite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリの性質

書籍『Writing A Tiny SQLite In Rust』(`book/`、mdbook、日本語) と、そのリファレンス実装 (`crates/chibisqlite/`) のモノレポ。`CREATE TABLE` / `INSERT` / `SELECT` / `UPDATE` / `DELETE` だけを持つ SQLite を Rust で書きながら、ページャ・B-tree・レコード形式・バイトコード VM を学ぶ構成になっている。

**書き出すファイルは本物の SQLite と互換のフォーマット**であり、自作エンジンが作った `.db` を `sqlite3` コマンドで開けることが本書の検証手段になっている。対応する SQL・型・式と、あえて実装しない機能は `book/src/appendix-a-spec.md` に確定済みなので、機能追加の可否はまずここを見る。

コードは書籍の引用元でもあるため、**実装の変更は書籍を壊しうる** (下記「書籍とコードの連動」)。

## コマンド

CI (`.github/workflows/ci.yml`) が実行するものが事実上の必須セット。

```sh
cargo test --all                                  # テスト全体
cargo fmt --all --check                           # フォーマット検査
cargo clippy --all-targets -- -D warnings         # lint (警告はエラー扱い)

cargo build --release && ./tools/verify/acceptance.sh target/release/chibisqlite  # L3 受入シナリオ
./tools/verify/check-anchors.sh                   # 本文の {{#include}} と実ソースの ANCHOR の整合
mdbook build book                                 # 書籍のビルド (cargo install mdbook --locked)
```

単一テストの実行 (統合テストはファイル単位で別クレート):

```sh
cargo test --test ch13_select                     # 章別テストファイル 1 つ
cargo test --test ch13_select select_all_columns  # その中の 1 関数
cargo test -p chibisqlite --lib                   # src 内のユニットテストのみ
```

その他:

```sh
cargo run -p chibisqlite -- mydata.db             # CLI。端末なら REPL、パイプならシェル
cargo run --example ch10_hand_written_bytecode -- demo.db  # 章の手書きサンプル (要 db ファイル)
./tools/verify/env.sh                             # sqlite3 の所在とバージョン確認
```

前提環境: Rust edition 2024 / rust-version 1.85、`sqlite3` コマンド 3.40.0 以降 (環境変数 `SQLITE3` で実行ファイルを差し替え可能)。依存クレートは REPL 用の `rustyline` のみで、それ以外は std だけで書く方針。

## アーキテクチャ

`crates/chibisqlite/src/` は下から積み上がる 4 層。上の層だけを見ても動作は追えないので、変更時は下層の契約を確認する。

- **ストレージ**: `header.rs` (100 バイトの DB ヘッダ、`PAGE_SIZE = 4096` 固定) → `pager.rs` (`Pager`: ページ単位の読み書きキャッシュ、`allocate` / `free_page` の freelist、`flush`) → `page.rs` (table b-tree ページのバイトレイアウト) → `varint.rs` / `record.rs` / `value.rs` (SQLite の varint・レコード形式・`Value` の比較規則) → `btree.rs` (`insert` / `delete` / `max_rowid` と走査用 `Cursor`)
- **SQL フロントエンド**: `tokenizer.rs` → `parser.rs` → `ast.rs` の `Statement` / `Expr`
- **実行エンジン**: `codegen.rs` (`compile(&Statement, &Schema) -> Vec<Instruction>`) → `vm.rs` (レジスタマシン。`Instruction` は約 20 命令、`Vm::step` が `StepResult::Row` / `Done` を返す)
- **統合**: `database.rs` (`Database`: `pager` + `schema` を持つ入り口) / `schema.rs` (ページ 1 の `sqlite_schema` を読み、保存済み `CREATE TABLE` 文を再パースして列を復元) / `shell.rs` (非対話シェル・ドットコマンド) / `repl.rs` (rustyline の対話ループ) / `main.rs` (`IsTerminal` で両者を振り分け)

SQL の流れは `Database::execute` (`src/database.rs:39`) に集約されている: `parser::parse` → `codegen::compile` → `Vm::step` を `Done` まで回し、`ResultRow` ごとに 1 行取り出す (本物の `sqlite3_step` と同じ形)。

**`CREATE TABLE` だけは例外**でバイトコードを経由せず、`Database::create_table` が直接ページを確保して `sqlite_schema` に 1 行挿入し `Schema::load` で読み直す。DDL を触るときはここを見る。

意図的な制約 (書籍の主題を絞るため。むやみに一般化しない): index b-tree なし・オーバーフローページなし (`MAX_PAYLOAD` 超のセルは扱えない)・トランザクション/ジャーナルなし・1 文 1 テーブル (カーソルは 0 番のみ)・`WHERE` は AND/OR 命令を持たず条件ジャンプの連なりにコンパイルされる。

## 書籍とコードの連動 (最重要)

本文はコードを写経ではなく**実ソースからの引用**で載せている。`src/*.rs` の `// ANCHOR: name` / `// ANCHOR_END: name` を、本文が `{{#include ../../crates/chibisqlite/src/xxx.rs:name}}` で取り込む。

- **ANCHOR コメントを消す・アンカー名を変える・アンカー範囲を移動すると書籍が壊れる**。`check-anchors.sh` が CI で落とす (mdbook は include の失敗を警告で素通りするため専用スクリプトがある)
- アンカーの中身を書き換えると本文の掲載コードも変わる。**コード変更時は該当章の説明文と食い違っていないか読み直す**。「実装 → 本文反映」をペアでコミットするのがこのリポジトリの運用 (git log 参照)
- 逆に、本文の説明を変えるだけならコードは触らない

## テストの方針

- **章別統合テスト `crates/chibisqlite/tests/chNN_*.rs` が各章の完了条件 (DoD)**。章番号でソースと対応する (ch02→header, ch03→pager, ch04→varint/record, ch05〜07→btree/page, ch08→tokenizer, ch09→parser/ast, ch10→vm, ch11→Cursor, ch12〜14→codegen/database, ch15→shell/repl/main)。新しい振る舞いを足すときは該当章のテストファイルに追加する
- ヘルパは `tests/common/mod.rs` を再利用する。`TempDb` (Drop で消える一時 DB、`sqlite3()` で本物の sqlite3 を実行)、`TempFile`、`sqlite3_on`
- 典型パターンは**本物の sqlite3 との出力突合**。`ch13_select.rs` の `assert_same_as_sqlite3(setup, query)` のように、同じ SQL を両者で実行して結果を比較する。fixture ファイルは持たず、テスト内で SQL を組み立てる
- 純粋ロジック (varint / record / header / page / tokenizer / parser / value) は同一ファイル内の `#[cfg(test)] mod tests`
- 検証は 4 層構成 (L1 章別テスト / L2 sqlite3 相互運用 / L3 受入シナリオ / L4 クリーンルーム)。層の定義と `acceptance.sh` が要求する CLI の契約は `tools/verify/README.md` にある。CLI の入出力仕様を変える変更はこの契約を壊さないか確認する
- 実装は TDD で進める (`src/lib.rs` のモジュールドキュメントに明記)

## コミット

gitmoji + Conventional Commit の日本語 subject に、Tidy First の `[BEHAVIORAL]` / `[STRUCTURAL]` を付ける (例: `♻️ refactor: [STRUCTURAL] シェルの 1 行の実行を execute_line に切り出した`)。実装コミットと書籍反映コミットは分ける。

---
> Source: [skanehira/chibisqlite](https://github.com/skanehira/chibisqlite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
