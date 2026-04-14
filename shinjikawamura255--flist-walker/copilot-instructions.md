## flist-walker

> このファイルはプロジェクト固有のエージェント指示です。ワークスペース共通の方針と矛盾する場合は本ファイルを優先します。

# AGENTS.md for FlistWalker

このファイルはプロジェクト固有のエージェント指示です。ワークスペース共通の方針と矛盾する場合は本ファイルを優先します。

## 1. 目的と範囲
- 目的: `fzf --walker` 相当の体験で、ファイル/フォルダを高速にファジー検索し、選択結果を実行またはオープンできる Rust ツールを開発する。
- スコープ In:
- Rust での本実装（GUI/CLI）
- `FileList.txt` / `filelist.txt` 優先読み込み（ルート直下のみ探索）
- File/Folder walker の分離
- fzf 互換クエリ（`'` 完全一致、`!` 除外、`^`/`$` の先頭末尾条件）
- 検索ハイライト、非マッチ非表示、複数選択と一括操作
- Windows 向けビルド運用（WSL/Linux から `x86_64-pc-windows-gnu` + mingw-w64 で完結）
- `docs/` の SDD + TDD 文書保守
- スコープ Out:
- 旧プロトタイプの機能追加
- ネットワークドライブ向け最適化
- 配布インストーラ作成

## 2. 現在の開発方針
- Rust 本実装を主軸に機能拡張と品質改善を行う。
- GUI/CLI の検索仕様・操作仕様の一貫性を維持する。
- Windows 実行を重視し、WSL/Linux から完結する GNU ビルド導線を維持する。
- 旧プロトタイプ資産は `prototype/python/` に保管し、原則として Rust 側へ集約する。

## 3. UI 応答性ポリシー（最優先）
- 最優先要件: UI を固めない。重い I/O や計算は UI スレッドで実行しない。
- MUST: index/search/preview/create-filelist はワーカー（スレッド）で処理し、UI はイベント駆動で反映する。
- MUST: 検索・インデックス・ファイルリスト作成など長時間処理中でも、入力・カーソル移動・スクロール操作を維持する。
- MUST: 処理中ステータス（例: `Searching...`, `Indexing...`, `Creating FileList...`）を UI で可視化する。
- SHOULD: 増分反映やメッセージ処理は 1 フレームの予算を設け、再描画詰まりを防ぐ。
- SHOULD: プレビューは遅延してもよいが、一覧操作の滑らかさを優先する。

## 4. 重要な制約・品質特性
- 対応環境: Windows/macOS/Linux の主要 OS。
- 性能: 10万件候補で検索応答 100ms 未満を目標。
- 品質: TDD を基本とし、主要機能は unit test で保証する。
- セキュリティ: 外部コマンド実行は配列引数で呼び出し、シェル展開依存を避ける。
- 運用:
- Windows 向け Rust ビルドは `scripts/build-rust-win.sh` / `scripts/build-rust-win-clean.sh` を利用し、`x86_64-pc-windows-gnu` + mingw-w64 で WSL/Linux から完結させる。
- GitHub Actions の tag push（`v*`）で各 OS 向け release build を実行し、draft release と asset upload を行う workflow を維持する。
- tag push 後は GitHub 上に作成された draft release を確認し、公開時は Codex から GitHub Release 本文を整えたうえで draft を本リリースへ切り替える運用とする。
- 当面の暫定運用として、macOS 配布物の notarization 確認は本リリース publish の前提条件にしない。notarization 環境が整うまでは publish を許容する。
- release asset の生成は `scripts/prepare-release*.sh|ps1` と `.github/workflows/release-tagged.yml` を基準に保守する。
- 配布アーカイブと standalone バイナリ向け sidecar asset には `LICENSE` / `THIRD_PARTY_NOTICES` を同梱し、依存ライセンス notice の欠落を防ぐ。
- macOS の `.app` bundle は notarization 用に `dist/` へ生成してよいが、GitHub Release へ添付する asset には含めない。
- release/tag 作業に入る前に、`skills/flistwalker-release-preflight/` を使って `rust/Cargo.toml` / `rust/Cargo.lock` / `CHANGELOG.md` / tag 名の version 整合を確認し、必要な更新を先に完了させる。

## 5. 実装ガードレール
- FileList 検出仕様（大文字/小文字、優先順、探索範囲）を変更する場合は `indexer` テストを先に更新する。
- 非同期処理は request_id で最新応答のみ採用し、古い応答で UI 状態を巻き戻さない。
- GUI でパス種別判定やプレビュー生成などの同期 I/O を描画ループに持ち込まない。
- 仕様演算子（`'`, `!`, `^`, `$`）の検索契約は後方互換を維持する。

## 6. ドキュメント/プロセス
- `docs/` に `REQUIREMENTS.md` / `SPEC.md` / `DESIGN.md` / `TESTPLAN.md` を配置。
- ID は `FR-###` / `NFR-###` / `CON-###` / `SP-###` / `DES-###` / `TC-###` を付与。
- SPEC は MUST/SHOULD で規範化し、TDD を徹底する。
- 仕様や設計を変更したら、同一変更で docs の該当箇所も更新する。
- 検証は `docs/TESTPLAN.md` の Validation Matrix に従って選択する。
- Rust 実装を変更した場合は最低限 `cargo test` を実行してから完了報告する。docs-only 変更は matrix 上の docs 手順で代替してよい。
- `rust/src/indexer.rs`、`rust/src/app/workers.rs`、`rust/src/app/mod.rs` のインデクシング経路を変更した場合は、Validation Matrix の VM-003 に従い、通常の `cargo test` に加えて以下の ignored perf テストを明示実行する。
- `cargo test perf_regression_filelist_stream_matches_v0123_reference_budget --lib -- --ignored --nocapture`
- `cargo test perf_walker_classification_is_faster_than_eager_metadata_resolution --lib -- --ignored --nocapture`
- 上記 perf テストは、FileList / Walker の初期インデクシング速度が基準実装より悪化していないことを確認する目的で使う。
- `rust/Cargo.toml` / `rust/Cargo.lock` / GitHub Actions / release script など依存関係や配布物に含まれる OSS 構成を変更した場合は、同一変更で `THIRD_PARTY_NOTICES.txt`、必要な `LICENSE` 同梱導線、関連 docs を更新し、`docs/OSS_COMPLIANCE.md` のチェックを実施してから完了報告する。
- `FLISTWALKER_UPDATE_FEED_URL` / `FLISTWALKER_UPDATE_ALLOW_SAME_VERSION` / `FLISTWALKER_UPDATE_ALLOW_DOWNGRADE` は開発・手動試験専用とし、`README.md`、`docs/RELEASE.md`、`.github/release-template.md`、GitHub Release 本文、CLI/GUI のユーザ向けヘルプなど配布物や公開向け文書へ記載してはならない。
- release asset 名、対象 OS、GitHub Release 導線を変更した場合は `docs/RELEASE.md`、`.github/release-template.md`、`AGENTS.md` を同一変更で更新する。
- `vX.Y.Z` の tag 作成、release note 整備、draft release publish を行う依頼では、先に `skills/flistwalker-release-preflight/` を実行し、version 更新漏れがあれば tag 作成前に修正する。
- リリースノート、`CHANGELOG.md`、GitHub Release 本文の更新は project-local skill `skills/flistwalker-release-notes/` の記法に従う。
- 上記の暫定運用中は、GitHub Release 本文の `Security` または `Known issues` に macOS 配布物が未 notarized である旨を明記する。

## 7. トレース（抜粋）
- FR-### → SP-### → DES-### → TC-###

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ShinjiKawamura255) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
