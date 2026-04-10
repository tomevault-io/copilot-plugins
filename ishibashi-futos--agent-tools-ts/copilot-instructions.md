## agent-tools-ts

> AIエージェントツールから利用されるユーティリティツールを集めたリポジトリです

## Overview

AIエージェントツールから利用されるユーティリティツールを集めたリポジトリです

## 技術仕様

- runtime: bun
- testing: `bun test`
- formatter: biome `bun run format`

## 作業ルール

- 日本語で応答する
- ドキュメント作成は日本語で行う
- コメントは日本語でつける
- コードは常にメンテナンス製とテスト容易性を最も重要視する
- タスクは常に逐次実行する。並列化は行わない
- 時間の節約・開発スループットよりも、確実とトレーサビリティを優先する
- 後方互換性や例外処理のために処理を複雑にせず、シンプルで唯一の正解のために DRY な実装を行う
- コミットメッセージは必ず次の形式にする: `<type>:<description>` の1行目 + 空行 + 本文
  - type は `fix` / `feat` / `docs` / `chore` のいずれかを使う
- PR作成には `gh` を使う
  - 本文にバッククォートやシェル展開される文字列を含む場合を想定し、`--body` 直書きは避けて常に `--body-file` を使う
  - 作成後は `gh pr view --json url,title,body` で本文崩れがないことを確認する

## Folder Rule

- `@src/ `- Bunのコード本体
  - `lib.ts` - 外部公開用の型定義・エクスポートなど
  - `utils/` - 内部で仕様するユーティリティ関数
  - `registory/` - tools に追加したツール定義を格納する
  - `sandbox/` & `security/` - Agentが実行するツール機能を制限するガードレール機構
  - `tools` - ツール機能の本体。機能ドメインごとにフォルダを分ける
- `@test/` - テストコードを格納する

## Definition of Done

- `scripts/sanity.sh` を実行し、型エラー・フォーマット・全件テストが全てパスすること

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ishibashi-futos)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ishibashi-futos)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
