## bootstrap-ai-tooling-all

> 最優先は docs/ai/PROJECT_CONTEXT.md。

# Claude Code Instructions（加美電機 / LAMP）

最優先は docs/ai/PROJECT_CONTEXT.md。

要点（最小）：
- 既存LAMP前提で、段階的改善。
- 可読性・デバッグ容易性・修正可能性を最優先。過剰設計しない。
- フレームワークは導入理由が明確な場合のみ可。
- ORMは状況次第。SQLの方が明確ならSQL優先。
- コメントは可能な限り残す（why/前提/業務的意味/例外）。
- 実装変更時は必ずドキュメント更新まで含める（更新不要なら理由明記）。
- **コーディング前**に影響範囲調査・テスト作成・実行を必須とする。**コーディング後**も同じテストを再実行して確認すること。テスト内容を明記。
- コード削除時は必須チェックリスト：
  1. 削除が意図的であることを明示的に確認
  2. 削除されたファイル/パス/関数をPR説明に列挙
  3. 理由と関連issue/commitへのリンクを提供
  4. ロールバック/復元手順を含める
  5. 削除をカバーするテストまたは手動検証手順をリスト
  6. 削除に対する明示的なレビュー承認を要求
- レビューは正しさ/整合性/セキュリティ/運用リスク重視。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kamo-git) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
