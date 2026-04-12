## cursor-rules

> Rule Name: task_structure

Rule Name: task_structure
Description: 1人開発向けタスク管理のディレクトリ構造に関するルール

# ディレクトリ構造
```
project_root/
├── requirements/          # 要件管理
│   ├── features/         # 機能要件
│   ├── non-functional/   # 非機能要件
│   └── constraints/      # 制約条件
│
├── tasks/                # タスク管理
│   ├── backlog/         # 未着手タスク
│   ├── in-progress/     # 進行中タスク
│   └── done/            # 完了タスク
│
└── docs/                # ドキュメント
    ├── design/          # 設計ドキュメント
    ├── implementation/  # 実装ドキュメント
    └── operation/       # 運用ドキュメント
```

# 要件管理のルール
- 機能要件は`requirements/features/`に保存
- 非機能要件は`requirements/non-functional/`に保存
- 制約条件は`requirements/constraints/`に保存
- 各要件ファイルはYAML形式で作成
- 要件の変更履歴を記録

# タスク管理のルール
- タスクは状態に応じて適切なディレクトリに移動
- タスクファイルはYAML形式で作成
- タスクの移動は必ず記録
- 完了したタスクは`done/`に移動

# タスクの状態遷移
```
backlog/ → in-progress/ → done/
```

# タスクファイルの命名規則
- 形式: `TASK-{連番}.yml`
- 例: `TASK-001.yml`, `TASK-002.yml`

# タスクファイルのテンプレート
```yaml
task:
  id: TASK-001
  title: "タスクのタイトル"
  description: "タスクの詳細説明"
  priority: 高/中/低
  estimate: "見積もり時間"
  dependencies: []
  status: "状態"
  created_at: "YYYY-MM-DD"
  updated_at: "YYYY-MM-DD"
  history:
    - date: "YYYY-MM-DD"
      from: "前の状態"
      to: "新しい状態"
      reason: "変更理由"
```

# タスクの完了条件
- コードの実装が完了していること
- テストがパスしていること
- ドキュメントが更新されていること
- コミットメッセージが適切に記述されていること

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yaguchi-1-9)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yaguchi-1-9)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
