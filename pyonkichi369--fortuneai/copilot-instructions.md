## fortuneai

> - **PRESIDENT** (別セッション): 統括責任者

# Agent Communication System

## エージェント構成

- **PRESIDENT** (別セッション): 統括責任者
- **boss1** (multiagent:0.0): チームリーダー
- **worker1,2,3** (multiagent:0.1-3): 実行担当

## あなたの役割

- **PRESIDENT**: @instructions/president.md
- **boss1**: @instructions/boss.md
- **worker1,2,3**: @instructions/worker.md

## メッセージ送信

```bash
./agent-send.sh [相手] "[メッセージ]"
```

## 基本フロー

PRESIDENT → boss1 → workers → boss1 → PRESIDENT

## 開発環境セットアップ

### 全体環境構築（初回のみ）

```bash
# 全エージェント環境を一括構築
./shell-scripts/setup.sh
```

### 各エージェント起動

```bash
# 最も簡単な方法（エイリアス使用）
president    # PRESIDENTエージェント
boss1        # boss1エージェント
worker1      # worker1エージェント
worker2      # worker2エージェント
worker3      # worker3エージェント
agents       # multiagentセッション全体表示

# スクリプト直接実行
./shell-scripts/start-president.sh
./shell-scripts/start-boss1.sh
./shell-scripts/start-worker1.sh
./shell-scripts/start-worker2.sh
./shell-scripts/start-worker3.sh

# 手動でセッションアタッチ
tmux attach -t president     # PRESIDENTセッション
tmux attach -t multiagent    # MultiAgentセッション
```

### エージェント間通信

```bash
# メッセージ送信
./shell-scripts/agent.sh [エージェント名] "[メッセージ]"

# 例
./shell-scripts/agent.sh boss1 "プロジェクト開始指示"
./shell-scripts/agent.sh worker1 "作業完了報告"
```

詳細な手順は [DEVELOPMENT.md](./DEVELOPMENT.md) を参照してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pyonkichi369)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pyonkichi369)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
