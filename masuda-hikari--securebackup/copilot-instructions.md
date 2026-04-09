## securebackup

> ﻿﻿﻿﻿﻿<!-- GOVERNANCE:BEGIN -->

﻿﻿﻿﻿﻿<!-- GOVERNANCE:BEGIN -->
<!-- 
  このセクションはオーケストレーターによって管理されています。
  手動編集は次回同期時に上書きされます。
  プロジェクト固有の内容は LOCAL セクションに記述してください。
-->

# プロジェクトガバナンス

継承: O:\Dev\CLAUDE.md → O:\Dev\Work\_orchestrator\CLAUDE.md → このファイル


## 進捗率の正しい認識（CRITICAL）

**進捗100% = 1000万円の現金収入を得た瞬間**

- 現在の収益: 0円
- 現在の進捗: **0%**
- コード完成・テスト通過・ビルド成功は進捗ではない
- 収益が発生して初めて進捗がカウントされる
- 批判的思考を維持し、楽観的な報告を避けること

参照: `O:\Dev\Work\_orchestrator\governance\rules\progress-definition-critical.md`

## 開発規範（共通）

### 言語設定
**日本語一律使用**: 応答/コメント/commit全て日本語

### 基本原則
- 抽象化: 共通処理100%/2箇所使用→即共通化/DRY
- 資源活用: 既存確認必須/改良優先

### 禁止事項
- 未テストコミット
- 認証情報ハードコード
- 脆弱性導入
- UTF-8以外のエンコーディング

## UI/UXエクセレンス（アプリ・ツール必須）

**参照**: O:\Dev\Work\_orchestrator\templates\uiux_excellence_guide.md

### Nielsen 10 Heuristics（必須）
1. システム状態の可視性（ローディング/成功/失敗表示）
2. 実世界との一致（ユーザーの言葉を使用）
3. ユーザーの制御と自由（戻る/取消/Undo）
4. 一貫性と標準（UI要素の統一）
5. エラー防止（バリデーション/確認ダイアログ）
6. 記憶より認識（選択肢表示/ヒント）
7. 柔軟性と効率性（ショートカット/カスタマイズ）
8. 美的でミニマル（余白/視覚階層）
9. エラー回復支援（解決策提示）
10. ヘルプとドキュメント（ツールチップ/FAQ）

### 必須対応
- ダークモード対応
- アクセシビリティ（コントラスト4.5:1以上、タップ44dp以上）
- 応答時間0.4秒以内（超過時はローディング表示）
- 日本市場配慮（かわいい系/可読性/品質感）

### プラットフォーム準拠
- iOS: Apple Human Interface Guidelines
- Android: Material Design 3
- Desktop: 各OS標準ガイドライン

## ミッション
**AIによる人間代替作業の実行と収益化**

## セッション終了時必須作業
1. 作業記録: logs/claude_runs に保存
2. Git Push: 全変更をコミット＆プッシュ
3. 次回引継ぎ: 未完了タスク・課題を明記

## Orchestrator統制
- logs/claude_runs にログ保存必須
- STATUS.md を最新状態に維持
- 失敗時は failure_classification を設定

<!-- GOVERNANCE:END -->
<!-- LOCAL:BEGIN -->
<!-- 
  このセクションはプロジェクト固有の内容です。
  オーケストレーター同期時も保持されます。
-->
# SecureBackup - プロジェクト固有設定

## 概要
<!-- プロジェクトの説明 -->

## 技術スタック
<!-- 使用技術 -->

## 固有ルール
<!-- プロジェクト固有のルール -->
<!-- LOCAL:END -->

<!--
Orchestrator管理タグ（編集禁止）
TEMPLATE_VERSION: 2.0
LAST_SYNC: 2026-01-18
ORCHESTRATOR_MANAGED: true
-->


## 統合効率化システム適用

**このプロジェクトは Claude Code 最大効率化システムの適用対象です。**

### 適用モジュール
- Context Cache: トークン削減30%
- Template Engine: コード再利用促進
- Revenue Planner: 収益逆算タスク生成
- Predictive Engine: 次タスク自動予測
- Code Learning Engine: コード品質自動進化

### 設定
詳細設定: `O:\Dev\Work\_orchestrator\config\max_efficiency_config.json`

### モデル選択ルール
- プランニング・詳細設計: **opus**
- 実装・ビルド・検証: **sonnet** (デフォルト)
- ログ記録・軽量タスク: **haiku**

### Revenue Planner
収益目標達成のためのタスクは `/revenue-tasks` で自動生成されます。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/masuda-hikari)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/masuda-hikari)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
