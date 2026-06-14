## zenn

> - **Zenn記事リポジトリ**: https://github.com/NEXT-STANDARD/zenn

# Zenn リポジトリ引き継ぎメモ

## リポジトリ情報

- **Zenn記事リポジトリ**: https://github.com/NEXT-STANDARD/zenn
- **MINATOプロトコル仕様書**: https://github.com/NEXT-STANDARD/minato-spec

## 進行中のシリーズ：MINATOプロトコル記事

AIエージェント間通信プロトコル「MINATO」の紹介記事シリーズ。
コントリビューター獲得・仕様の周知・実装者の確保が目的。
ターゲット読者：AIエージェントを使っている・興味がある開発者。

### 記事1（執筆済み・非公開）

- **ファイル**: `articles/minato-protocol-why-spec-first.md`
- **タイトル**: `MCP・A2Aの死角——デバイスをまたぐAIエージェント通信プロトコルを作っている話`
- **文字数**: 約6400文字
- **状態**: `published: false`（公開タイミングは本人判断）
- **内容**:
  - なぜ実装でなく仕様書を先に公開したのか
  - MCP・A2A・ANPとの比較表
  - BLE Mesh + Nostrのデュアルトランスポート設計
  - インターネット不要・災害時ユースケース（日本発であることも言及）
  - 4段階信頼レベル（Apprentice/Partner/Lieutenant/Alter Ego）
  - AIエージェントがAIエージェントを使役する未来（少し怖い話）
  - コントリビューター募集CTA

### 記事候補リスト（2〜5本目）

| # | タイトル案 | 内容 | 優先度 |
|---|-----------|------|--------|
| 2 | 「弟子」から「分身」まで——AIエージェント間の信頼を4段階で設計する | 信頼レベルの設計思想。単独でも広まりやすい。 | 高 |
| 3 | BLE Mesh × Nostr の二重化——近距離と長距離を1プロトコルで扱う設計 | トランスポート層の技術選定と意思決定 | 高 |
| 4 | MINATOのメッセージ型7種——プロトコル仕様を全部公開する | 実装者向け仕様書回。スキーマ・コードを多用。 | 中 |
| 5 | MINATOプロトコルを一緒に実装しませんか | 現状・求める実装・参加方法。CTA記事。 | 中 |

### MINATOプロトコル概要（記事執筆時の参照用）

- **設計思想**: デバイス間（スマホ↔スマホ）のAIエージェント通信。中央サーバーなし・アカウント不要。
- **トランスポート**: BLE Mesh（近距離・インターネット不要）+ Nostrリレー（遠距離・分散）の自動切替
- **BLE Meshの到達距離**: 1ホップ10〜100m、最大127ホップ（理論上1km以上）
- **メッセージ型**: 7種（HANDSHAKE/MESSAGE/REQUEST/RESPONSE/ACK/REVOKE/PING/LOG）
- **信頼レベル**: Apprentice（毎回確認）/ Partner（実行前確認）/ Lieutenant（高リスクのみ確認）/ Alter Ego（全自動）
- **多言語対応**: ペイロードに原文＋翻訳を両方格納、プロトコルレベルで言語透過
- **セキュリティ**: Noise Protocol Framework + Ed25519署名
- **iOS実装**: 別リポジトリで非公開開発中（記事では触り程度にとどめる）
- **ライセンス**: MIT

### 競合・協業プロジェクト

| プロジェクト | 関係 | URL |
|---|---|---|
| MCP（Anthropic） | 補完関係（レイヤーが違う） | https://modelcontextprotocol.io |
| A2A（Google） | 部分競合・補完（クラウド前提） | https://a2a-protocol.org |
| ANP | 思想が近い・連携候補 | https://agent-network-protocol.com |
| NANDA（MIT） | 思想が近い・連携候補 | https://www.media.mit.edu/projects/mit-nanda/overview/ |
| Nostrコミュニティ | MINATOのトランスポート層として使用中 | https://nostr.com |

## 記事の執筆スタイル

- カジュアルな口語体（論文調にしない）
- 具体的なシナリオで冒頭をフック
- ペイン→背景→設計→ゲインの流れ
- 比較表を使って視覚的に整理
- 「怖い話」要素で未来感を出しつつ、設計で解決していることを示す
- 5000〜7000文字程度
- 末尾は必ずリポジトリURLとコントリビューター募集CTA

## その他メモ

- 全過去記事は `published: false`（2026-04-25 に一括非公開化済み）
- 公開時は frontmatter の `published: false` → `true` に変更してpushするだけ

---
> Source: [NEXT-STANDARD/zenn](https://github.com/NEXT-STANDARD/zenn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
