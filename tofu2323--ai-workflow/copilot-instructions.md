## ai-workflow

> **このワークフローでは、AIは段階的に作業を進め、各ゲートで必ず人間の承認を待ちます。**

# AI-人間協働プロダクト開発ワークフロー

## ⚠️ 重要な動作原則

**このワークフローでは、AIは段階的に作業を進め、各ゲートで必ず人間の承認を待ちます。**

- ❌ **禁止**: 一度にすべてのフェーズを自動実行すること
- ✅ **推奨**: 各フェーズごとに評価→人間承認→次へ進む

## 🤖 エージェントの自動選択

ユーザーがエージェントを指定しなくても、タスクの内容から最適なエージェントを自動的に選択して提案します：

| ユーザーの要求 | 推奨エージェント | 理由 |
|---------------|-----------------|------|
| 「インタビュー結果を整理して」 | 1-issue-analyzer | JTBDIssue作成が必要 |
| 「SYS-XXXをチェックして」 | validation-checker | ゲート基準の評価が必要 |
| 「StoryをINVEST基準で評価」 | 3-story-splitter | Story分割・評価が必要 |
| 「設計書を作成して」 | 4-design-generator | 技術設計が必要 |
| 「リリース計画を立てて」 | 5-scope-planner | Scope定義が必要 |

AIは作業開始前に「〜エージェントを使用します」と宣言してから実行します。

## 概要
このプロジェクトは、Linear をタスク管理のハブとして、Model Context Protocol (MCP) 経由でAIがLinearデータへアクセス・更新できる環境を提供します。すべての活動をタスクとして扱い、AI と人間が協力して製品開発を進めるための運用マニュアルに基づいています。

## 基本原則

### 1. タスク化を徹底
- 調査・デザイン・設計・実装・レビューなど、すべての活動をタスクとして管理
- タスクは 30 分〜数時間で終わる粒度を目安
- 完了すれば Story や Scope が前進するかを確認

### 2. 段階的リファインメント
- 粗い課題 (Issue) から始め、定期的なバックログリファインメントで Story → Scope → タスクへと成熟
- Story は初期段階では粗くてよく、スプリントが近づくにつれて INVEST を満たすように分割・詳細化

### 3. AI と人間の協働
- テンプレートに沿って AI が自動生成・チェック・分割を行い、人間は優先順位付けや承認・リファクタリングに集中
- AI が生成した成果物を人間が評価し、必要ならプロンプトや基準を更新

## フェーズ構成

### 2.1 リサーチ & Issue収集
- **目的**: ユーザー課題や市場ニーズを幅広く収集
- **入力**: 調査資料・インタビュー記録
- **出力**: 課題定義タスク (issue_id) - JTBD形式

### 2.2 Issue整理 & Story化
- **目的**: 集めた課題をユーザー価値ベースのStoryに変換
- **入力**: Issueタスク
- **出力**: 粗いStoryタスク (story_id)

### 2.3 Storyリファインメント & 設計
- **目的**: StoryをINVESTとSPIDR基準で成熟させ、設計を進める
- **入力**: 粗いStoryタスク
- **出力**: 詳細Storyタスク、設計タスク

### 2.4 Scope定義（リリース戦略含む）
- **目的**: Storyを束ねてリリース単位にまとめ、リリース段取りを定める
- **入力**: 詳細Storyタスク、設計タスク
- **出力**: Scopeタスク (scope_id)

### 2.5 実装 & 検証
- **目的**: Scopeに含まれるStoryをコードやドキュメントとして実装
- **入力**: Scopeタスク、Story/設計タスク
- **出力**: 実装タスク（各種成果物）

### 2.6 効果測定 & フィードバック
- **目的**: Scope公開後の成果を評価し、改善点をバックログに戻す
- **入力**: Scopeタスク、実装タスク、ユーザーデータ
- **出力**: 改善Issueタスク、レトロスペクティブ文書

## ゲート管理（人間承認必須）

各ゲートではAIが評価を行った後、**必ず人間のレビューと承認が必要**です：

1. **Gate 1 – Issue → 粗Story**: 
   - AI評価: Issue が JTBD形式で明確に記述され、証拠とKRとの紐づけがある
   - 人間承認: JTBD形式の妥当性、証拠の十分性、Story変換方針の確認

2. **Gate 2 – 粗Story → 詳細Story**: 
   - AI評価: Story が INVEST を満たす粒度まで詳細化されている
   - 人間承認: INVEST基準の達成度、分割の必要性、受け入れ条件の妥当性

3. **Gate 3 – Story/設計 → Scope**: 
   - AI評価: Story がリリース単位でグループ化され、リリース段取りが定義されている
   - 人間承認: リリース戦略の妥当性、リスク評価、Feature Flag設定の確認

4. **Gate 4 – Scope → 実装**: 
   - AI評価: Scope に紐づくタスクが DoR (Definition of Ready) を満たしている
   - 人間承認: タスク粒度（30分〜2時間）の確認、見積もりの妥当性、依存関係の確認

5. **Gate 5 – 実装 → 公開**: 
   - AI評価: タスクが完了し、CI/CDテストに合格し、Scopeのリリース条件を満たしている
   - 人間承認: 最終的なリリース判断、段階的リリースの選択、ロールバック準備の確認

## タスク分割ガイドライン

- **サイズ基準**: タスクは 30 分〜2 時間程度で完了できる粒度
- **価値基準**: タスクを完了すれば Story または Scope の前進に寄与
- **DoR/受け入れ基準**: 各タスクには完了条件と確認手順を明記
- **不明点の分離**: 調査や検証が必要な項目は Spike タスクとして独立
- **依存関係の明示**: 他タスクやチームメンバーへの依存を記述

## INVEST基準
- **Independent（独立）**: 他のストーリーから独立して実装可能
- **Negotiable（交渉可能）**: 詳細は開発時に調整可能
- **Valuable（価値がある）**: ユーザーやビジネスに価値を提供
- **Estimable（見積もり可能）**: 工数の見積もりが可能
- **Small（小さい）**: スプリント内で完了可能
- **Testable（テスト可能）**: 受け入れ条件が明確でテスト可能

## SPIDR分割観点
- **Spikes**: 未知の技術調査
- **Paths**: 複数の利用経路を区別
- **Interfaces**: UI と API のような異なるインターフェース
- **Data**: データの規模やパフォーマンスで区別
- **Rules**: バリデーションや権限など明確なルール

## ツール統合

### 必須ツール
- **Linear**: タスク管理のハブ（MCP経由でAIアクセス）
- **GitHub/GitLab**: コード管理
- **CI/CD**: 自動テスト・デプロイ

### 推奨ツール
- **Figma**: UI/UXデザイン
- **Notion/Confluence**: ドキュメント管理
- **Datadog/BigQuery**: データ分析
- **LaunchDarkly**: Feature Flag管理

### AI支援ツール
- **Claude Code**: コード生成・レビュー
- **Cursor/Devin**: 開発支援
- **Grok**: 分析・提案

## Linear MCP設定

Linear MCPサーバーは以下の機能を提供:
- Issues の作成・更新・検索
- Projects の管理
- Cycles の管理
- Comments の作成・更新

環境変数:
```
LINEAR_API_KEY=your_linear_api_key
```

## 使用方法

1. **新規プロジェクト開始時**
   - リサーチ結果から Issue を作成
   - Issue を Story に変換
   - Story をリファインメントして詳細化

2. **スプリント計画時**
   - Story を INVEST 基準で評価
   - SPIDR 観点で分割
   - Scope を定義してリリース計画を策定

3. **実装時**
   - タスクを 30分〜2時間単位に分割
   - AI にコード生成を支援させる
   - CI/CD でテストを自動実行

4. **リリース後**
   - メトリクスを分析
   - レトロスペクティブを実施
   - 改善 Issue を起票

## 使用例（AI対話）

### 段階的な進め方（推奨）
```
# Step 1: Issue評価
"SYS-181をGate 1基準で評価して"
→ AIが評価結果を提示
→ 人間が承認/修正指示

# Step 2: Story変換
"承認されたのでStoryに変換して"
→ AIがStory案を作成
→ 人間がINVEST基準でレビュー

# Step 3: Story詳細化
"Story SYS-195をINVEST基準で評価して"
→ AIが評価と改善案を提示
→ 人間が分割/詳細化を判断

# Step 4: Scope定義
"関連StoryをまとめてScope定義して"
→ AIがリリース計画を作成
→ 人間がリスクと戦略を承認

# Step 5: タスク分割
"承認されたScopeをタスクに分割して"
→ AIが30分〜2時間単位でタスク化
→ 人間が粒度と見積もりを確認
```

### 一括実行の禁止例
```
# ❌ 悪い例（一気に全部やってしまう）
"SYS-181をワークフローに則って最初から最後まで全部やって"

# ✅ 良い例（段階的に進める）
"SYS-181をGate 1基準で評価して、結果を見せて"
```

## 人間承認プロセス

### 承認フロー
1. **AIが評価・提案を実施**
   - 各フェーズの作業を実行
   - 評価結果をレポート形式で提示
   - 推奨アクションを提案

2. **人間がレビュー・判断**
   - 評価結果の確認
   - リスクの検討
   - 承認/修正/却下の判断

3. **承認後に次フェーズへ**
   - 人間の承認があって初めて次に進む
   - 修正指示があれば、AIが対応
   - 却下の場合は、代替案を検討

### タイムアウト設定
- Gate 1-2: 24時間
- Gate 3: 6時間
- Gate 4: 12時間
- Gate 5: 1時間

### エスカレーション
タイムアウト時は自動的に上位承認者へエスカレーション

## 参考資料

本ワークフローは以下の資料に基づいています：

1. **Linear Docs** - タスク管理とMCP統合
   - [Editor](https://linear.app/docs/editor)
   - [MCP Server](https://linear.app/docs/mcp)

2. **アジャイル手法**
   - [INVEST基準](https://en.wikipedia.org/wiki/INVEST_(mnemonic))
   - [SPIDR分割手法](https://ascendle.com/ideas/spidr-an-alternative-method-for-splitting-user-stories/)
   - [タスク分割ガイド](https://www.pluralsight.com/resources/blog/guides/break-down-agile-user-stories-into-tasks-and-estimate-level-of-effort)

3. **開発プラクティス**
   - [BDD手法](https://blog.ifs.com/2022/12/behavior-driven-development-bdd-approach-for-user-story-writing/)
   - [Agile UX Design](https://www.seamgen.com/blog/agile-ux-design-process)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tofu2323)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tofu2323)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
