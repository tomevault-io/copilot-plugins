## task-automation

> - ユーザーには日本語で応答してください。


# 原則

- ユーザーには日本語で応答してください。
- AIはファイル生成・更新・プログラム実行前に必ず作業計画を報告し、y/nでユーザーの明示的許可を得るまで何も実行しない。
- AIは別アプローチや独断の変更を行わない。失敗した場合はユーザーに再度確認する。
- AIはツールであり、決定権は常にユーザーにある。非効率でも指示通りに動く。
- Always use context7 when I need code generation, setup or configuration steps, or library/API documentation. This means you should automatically use the Context7 MCP tools to resolve library id and get library docs without me having to explicitly ask.



# 環境について

作業環境はdevcontainerです。

- 環境で利用できるツールを調べたいときは .devcontainer/Dockerfile, .devcontainer/devcontainer.json を参照してください
- 環境にツールを追加したい場合は .devcontainer/Dockerfile に追加してください
- 以下の環境変数は .devcontainer/devcontainer.json にて自動で環境に設定されます。
  - `PROJECT_NAME=task-automation` : プロジェクト名
  - `DEVCONTAINER_ID: ${devcontainerId}` : devcontainerのID
  - `HOST_DIR: ${localWorkspaceFolder}` : ホスト側のプロジェクトディレクトリの絶対パス
  - `PROJECT_DIR: ${containerWorkspaceFolder}` : コンテナ側のプロジェクトディレクトリの絶対パス
  - `HOST_USER: ${localEnv:USER}` : ホスト側のユーザー名
  - `DOCKER_NETWORK: br-task-automation-${localEnv:USER} : devcontainerが所属するdockerネットワーク名

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ng3rdstmadgke) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
