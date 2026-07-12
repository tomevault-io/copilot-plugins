## tech-article-fetcher

> アーキテクチャ・技術スタック・仕様の詳細は `README.md` を参照。

アーキテクチャ・技術スタック・仕様の詳細は `README.md` を参照。

# 開発スタイル

TDD で開発する（探索 → Red → Green → Refactoring）。
KPI やカバレッジ目標が与えられたら、達成するまで試行する。
不明瞭な指示は質問して明確にする。
詰まったら黙って試行し続けず、現状・原因の仮説・選択肢を短く報告して指示を仰ぐ（長時間の無言作業をしない）。
作業が完了したらコミット・push・PR起票まで行う。
PR を切る前に CI と同じチェックをローカルで全て通す（commit-and-pr スキル参照）。CI が落ちたら fix-ci スキルで対応する。
main は保護ブランチのため直接プッシュ不可。
必ずmainの最新を取り込んでからブランチを切って作業し、PR 経由でマージする。

# ブランチ運用

```bash
git switch -c <branch-name>                  # ブランチを作成して切り替え
# 変更・コミット
git push -u origin <branch-name>             # リモートにプッシュ
gh pr create --title "<title>" --body "<body>"  # PR起票（自動マージ有効）
```

# コード設計

- 関心の分離を保つ・状態とロジックを分離する
- コントラクト層（API/型）を厳密に定義し、実装層は再生成可能に保つ
- 静的検査可能なルールはプロンプトではなく linter か ast-grep で記述する

# コマンド

```bash
pip install -e ".[dev]"   # インストール
python -m src.main        # ローカル実行（.env 要設定）
pytest tests/ -v          # テスト
ruff check src/ tests/    # lint
mypy src/                 # 型検査
```

dashboard（`dashboard/` 以下、pnpm）:

```bash
cd dashboard
pnpm install              # インストール
pnpm dev                  # ローカル実行
pnpm lint                 # lint（biome）
pnpm typecheck            # 型検査（tsc）
pnpm test                 # テスト（vitest）
pnpm build                # ビルド
```

# 開発環境

- devcontainer への環境カスタマイズ（シェル設定・ツール追加など）は
  `devcontainer.json` / `postcreate.sh` に永続化する。
  リビルドで消える場所（コンテナ内ホームへの直接変更など）に書かない。
- GitHub の secrets / PAT は Claude からは設定できない。
  必要な secret 名と権限を具体的に提示してユーザーに登録を依頼する。

---
> Source: [ht311/tech-article-fetcher](https://github.com/ht311/tech-article-fetcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
