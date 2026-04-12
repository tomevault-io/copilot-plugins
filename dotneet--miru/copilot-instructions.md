## miru

> このプロジェクトは、以下の方針を最優先とします。

このプロジェクトは、以下の方針を最優先とします。

- コードは常にクリーンに保つ（可読性・一貫性・簡潔さを重視）
- 小さく頻繁にリファクタリングし、負債の蓄積を避ける
- AIコーディングエージェントを積極的に活用し、作業の質と速度を高める
- ドキュメントを最新に保つ。実装とドキュメントの乖離を最小にします。

## プロジェクト概要
MiroクローンMVPを構築する。無限キャンバス上で図形を作成・編集し、WebSocketでリアルタイム同期する共同編集ボードを提供する。UIは設計仕様に忠実に実装し、プロトコル仕様を単一ソースとしてフロント/バックの型を同期する。

## リポジトリ構成
- `frontend/`: Webクライアント（React Konva / Tailwind / shadcn/ui / Zustand）
- `backend/`: WebSocketコラボサーバ（Go + gorilla/websocket）
- `specs/`: 仕様の単一ソース（計画/UI/プロトコル/テスト/要件）
- `impl-plan.md`: 実装フェーズ計画

## 常に理解しておくべきこと（エージェント向け）
- 仕様の単一ソースは `specs/` 配下。特に `specs/03_protocol.md` が通信仕様の基準。
- MVPスコープは図形ツール（矩形/楕円/線/矢印/テキスト/付箋）、選択/移動/リサイズ/回転/削除、スタイル編集、パン/ズーム、WebSocket同期、プレゼンス、`boardId` 入室。
- MVP対象外はコメント/カード、認証/権限管理、永続化、高度なOT/CRDT、複数選択/グループ化/ロック、Undo/Redo（UIプレースホルダのみ）。
- 通信は WebSocket 上の JSON。serverSeq/clientSeq 管理、ギャップ検出、LWW 競合解決、レート制限を遵守する。
- UI/UXは `specs/02_ui_ux_design.md` のトークンとコンポーネント規約に従い、勝手なデザイン変更をしない。

## コーディングルール

 - 500行を超える場合はファイルを分割する。
 - 設計で示されたディレクトリ構造に従う。
 - リファクタリングは常に検討する。

## テスト方針（要点）
- テストピラミッドに従い、単体 → 結合 → E2E の順で厚くする。
- フロントは状態管理/プロトコルバリデーション/ジオメトリ計算を単体テストで担保する。
- バックエンドはオペレーション適用、バリデーション、シーケンス番号の単体テストを優先する。
- WebSocket統合テスト、フロント統合テスト、PlaywrightによるE2Eを用意する。
- 性能（描画60FPS/100オブジェクト、ネットワーク）と負荷（同時接続）のテストを行う。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dotneet)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dotneet)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
