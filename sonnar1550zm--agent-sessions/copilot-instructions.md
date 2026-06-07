## agent-sessions

> - このプロジェクトでコード、設定、リソースなどアプリ動作に関わる変更を行った場合は、作業完了前に必ずアプリを再作成し、起動中のアプリを再起動する。

# 原則として日本語で回答

# Web検索を積極的に利用

# ユーザーに迎合せず、正確性・論理性・意思決定品質を最優先

# Agent Sessions プロジェクト運用ルール

- このプロジェクトでコード、設定、リソースなどアプリ動作に関わる変更を行った場合は、作業完了前に必ずアプリを再作成し、起動中のアプリを再起動する。
- `AGENTS.md` のみを変更した場合は、アプリの再作成・再起動は不要。
- 標準手順は `swift test` を必要に応じて実行し、その後 `./scripts/build-app.sh` で `Agent Sessions.app` を再作成する。
- この環境では SwiftPM がユーザー領域の ModuleCache に書き込むため、`./scripts/build-app.sh` は最初から承認付きで実行する。サンドボックス内で失敗してから再実行する手順にしない。
- 再作成後は起動中の `Agent Sessions` を終了し、この checkout 直下の `Agent Sessions.app` を起動し直す。起動後のプロセス確認までは不要。
- 最終報告では、再作成と再起動が完了したか、または不要・実行できなかった理由を明記する。

---
> Source: [Sonnar1550ZM/Agent-Sessions](https://github.com/Sonnar1550ZM/Agent-Sessions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
