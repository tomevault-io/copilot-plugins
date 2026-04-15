## google-calendar-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commit and Pull Request Workflow

**重要**:
- コミットを作成する時は`commit-creator`エージェントを利用してください。このエージェントは変更内容を自動的に分析し、適切なコミットメッセージを生成します。
- プルリクエストを作成する時は`pr-creator`エージェントを利用してください。このエージェントはコミット履歴から適切なPR説明を生成します。

## Code Quality Checks

**TypeScriptコードや関連するファイルを変更した場合**、プルリクエストを作成する前に以下のチェックを実行してください:

1. **テストの実行**: `npm run test:run`
   - すべてのユニットテストが通過することを確認
   - テストが失敗した場合は修正してから進める
   - 注: `npm test`はwatchモードで動作するため、CI環境では`test:run`を使用

2. **ビルドの実行**: `npm run build`
   - TypeScriptのコンパイルエラーがないことを確認
   - ビルドが成功することを確認

3. **型チェック**: `npm run typecheck`
   - 型エラーがないことを確認
   - TypeScriptの型安全性を維持

4. **リンターの実行**: `npm run lint`
   - コーディング規約に従っていることを確認
   - 自動修正可能な問題は `npm run lint:fix` で修正

5. **フォーマッターの実行**: `npm run format`
   - コードスタイルを統一
   - Prettierによる自動整形

**推奨フロー**:
```bash
# すべてのチェックを順番に実行
npm run typecheck && npm run lint && npm run format && npm run test:run && npm run build
```

これらのチェックに失敗がある場合は、プルリクエストを作成せずに問題を修正してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ham0215)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ham0215)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
