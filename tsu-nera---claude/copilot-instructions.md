## claude

> - システムコンテキストの日付から曜日を判断しない。`date +%A` で確認すること

## General Tips
- システムコンテキストの日付から曜日を判断しない。`date +%A` で確認すること
- SSH接続が失敗した場合は、数秒待ってからリトライすること（一時的なネットワーク障害の可能性がある）

## Development Tips
- `gh issue create` / `gh issue comment` / `gh pr create` の本文末尾には必ずセッションIDフッターを付与すること:
  ```
  ---
  🤖 Claude Code session: `$CLAUDE_SESSION_ID`
  ```

## Communication Strategy
- 英語で思考、日本語で応答
- 表はmarkdownテーブル形式（`| col | col |`）で出力する。罫線文字（┌─┬─┐等）は使わない

- リファクタリングで不要なコードはコメントアウトやdeprecatedではなく完全に削除
- コーディング規約: `~/.claude/docs/coding-guidelines.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tsu-nera) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
