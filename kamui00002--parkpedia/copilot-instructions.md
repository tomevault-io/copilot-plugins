## parkpedia

> Codex が ParkPedia を触るときの**最短導線**。詳細は `docs/` 配下の各ドキュメントを参照。

# ParkPedia 開発ガイド ⭐️

Codex が ParkPedia を触るときの**最短導線**。詳細は `docs/` 配下の各ドキュメントを参照。

## プロジェクト概要
| 項目 | 値 |
|---|---|
| アプリ | 公園情報を共有する React Native アプリ（Expo） |
| 主要 SDK | Firebase (Auth / Firestore / Storage), AdMob ☁️ |
| 起動/構成/リンク集 | @README.md |

## ドキュメント導線（まず見る場所）
| ドキュメント | 用途 | マーク |
|---|---|---|
| @docs/DEVELOPMENT_KNOWLEDGE_BASE.md | 運用・審査・トラブルシュート集約 | ⭐️ |
| @docs/CRITICAL_SECURITY_ADDITIONS.md | セキュリティ必須対応事項 | ⭐️ |
| @docs/DATA_RETENTION_POLICY.md | データ保持ポリシー | ☁️ |
| @docs/PRIVACY_DATA_INVENTORY.md | プライバシーデータ一覧 | ☁️ |
| @docs/GITLEAKS_SETUP.md | Secret スキャン設定 | ☀️ |
| @docs/ARCHIVE.md | 過去メモの統合アーカイブ（旧 md 全内容） | — |
| @docs/guides/ | 手順書ディレクトリ | — |
| @docs/reports/ | レポート・調査結果 | — |

## Firebase ルール運用（最低限）
- **Firestore**: `firestore.rules` を Firebase Console > Firestore Database > ルール に貼り付けて公開 ☁️
- **Storage**: `storage.rules` を Firebase Console > Storage > ルール に貼り付けて公開 ☁️

## よくある落とし穴（超要点）
- **Firestore の `list` では `resource.data` を使えない**（`allow read` を安易に書くと詰む）
- **Firestore と Storage のルールは貼り付け先を絶対に混同しない**
- **期限付きのテストモードルール（`request.time < ...`）を残さない**

## ファイルマーキング規則
使用したファイル（.md / .json / .js / .ts）に絵文字タグを付けて機能を示すこと：
- ⭐️ = プロジェクト固有パターン
- ☀️ = 自動化・ワークフロー
- ☁️ = 外部連携・プロセス（Firebase / AdMob / App Store）

詳細手順・過去事例は @docs/DEVELOPMENT_KNOWLEDGE_BASE.md と @docs/ARCHIVE.md を参照。

---
> Source: [kamui00002/ParkPedia](https://github.com/kamui00002/ParkPedia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
