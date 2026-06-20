## codex-startuptools-new-linux

> このリポジトリは、**Codex ネイティブな移植プロジェクト**として運用します。

# AGENTS.md

## Codex 運用モデル

このリポジトリは、**Codex ネイティブな移植プロジェクト**として運用します。  
また、**Codex を主たる開発環境および実装エージェント**として使用します。

主な目的:

- 参照Windows版リポジトリを分析する
- 再利用可能な設計、運用、検証パターンを保持する
- Codex に適したものだけを再実装する
- Codex を前提とした開発フローで移植先リポジトリを育てる
- Linux ローカル起動、`/home/kensan/Projects` 配下の登録プロジェクト候補、Codex only を前提にする
- Supervisor 設定は登録プロジェクト候補へ適用可能にする

優先実行順:

1. monitor
2. build
3. verify
4. improve

基本ルール:

- 小さく、戻しやすい変更を優先する
- 検証していない機能互換を主張しない
- ファイルを移す前に、まず概念を翻訳する
- 元リポジトリ特有の前提は移植ノートに残す
- 実行可能な振る舞いを移すときはテストを追加する
- SSH / Claude / Copilot 起動機能は実装対象外とする

停止条件:

- 同じ種類のブロッカーが 3 回連続で発生
- 外部依存が不明で安全に進められない
- Claude 専用ランタイムに依存し、Codex で安全な代替が用意できない

成果物の最低基準:

- 各移植機能に以下を持たせる
- 目的
- 元機能との対応関係
- Codex 向けの変換メモ
- 検証方法

---
> Source: [Kensan196948G/Codex-StartUpTools-New-Linux](https://github.com/Kensan196948G/Codex-StartUpTools-New-Linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
