## gitws

> 作業を行う前に、設計、検討内容、作業リストを ai-docs/works/ 配下に必ず書いて作業を開始してください


# 作業ルール（必ず守ること）

## 事前準備（作業前に必ず実行）

### 1. 作業ドキュメントの作成

作業を行う前に、設計、検討内容、作業リストを ai-docs/works/ 配下に必ず書いて作業を開始してください

**フォーマット**: `yyyymmdd-<workname>.md`

### 2. ブランチ管理

作業にファイル更新が入る場合は、main ブランチからブランチを切って作業してください

```bash
git checkout main
git pull origin main
git checkout -b feature/<feature-name>
```

## 実装時のルール

### 設計ドキュメントの参照

- 実装については、@ai-docs/designs/design.md を参照して進めてください
- 実装方針に変更があった場合は必ず、@ai-docs/designs/design.md, @ai-docs/designs/tasks.md も更新してください

### タスク管理

- 実装は @ai-docs/designs/tasks.md を見て進める
- リストの更新(チェック)も必ず行って同期してください

### テスト作成（必須）

- **新しい機能を実装する際は、必ずテストも同時に作成してください**
- 単体テストは各モジュール内の`#[cfg(test)]`モジュールに記述
- 統合テストは`tests/`ディレクトリに配置
- エラーハンドリング（`Result<T, E>`）も含めてテストケースを作成
- 非同期処理のテストは`#[tokio::test]`を使用

## 完了時のルール

### 品質チェック（必須）

実装が完了したら、以下を必ず実行してください:

```bash
# 1. コードフォーマット
cargo fmt

# 2. 警告チェック
cargo clippy --all-targets -- -D warnings

# 3. テスト実行
cargo test
```

### 余計なブランチや worktree が残っていないか確認

- test の実行により、余計なブランチや worktree が残っていないか確認し、おかしい場合はテストの挙動を修正してください

### ドキュメントの更新

- 全ての作業完了後に、task.md を更新すること
- README.md README.ja.md に現在の実装との乖離があれば更新すること

### PR 作成

完了したら、gh コマンドで PR を作成して報告してください

```bash
gh pr create --title "タイトル" --body "説明"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/morooka-akira) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
