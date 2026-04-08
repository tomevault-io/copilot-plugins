## cashchangersimulator

> This document is for AI agents (like Antigravity) working on this repository. It contains project-specific knowledge that should be prioritized.

# CashChangerSimulator Agent Guide / エージェントガイド

This document is for AI agents (like Antigravity) working on this repository. It contains project-specific knowledge that should be prioritized.
このドキュメントは、本リポジトリで作業する AI エージェント（Antigravity など）向けのもので、優先すべきプロジェクト固有の知識が含まれています。

## 1. Critical Locations / 重要な場所

- **Obsidian Vault**: The path to the external knowledge base (Obsidian Vault) is stored in [vault_path.txt](file:///C:/Users/ITI202301003_User/source/repos/w-red/CashChangerSimulator/LocalSettings/vault_path.txt). Always read this file to find where to update documentation (e.g., `WarningResolutions.md`).
- **Obsidian Vault**: 外部ナレッジベース（Obsidian Vault）へのパスは [vault_path.txt](file:///C:/Users/ITI202301003_User/source/repos/w-red/CashChangerSimulator/LocalSettings/vault_path.txt) に格納されています。ドキュメント（`WarningResolutions.md` など）を更新する際は、必ずこのファイルを読み取って場所を確認してください。
- **Configuration**: The project root `config.toml` is a link source, but the running app uses the copy in `bin/Debug/...`.
- **設定ファイル**: プロジェクトルートの `config.toml` はリンク元ですが、実行中のアプリは `bin/Debug/...` 内のコピーを使用します。

## 2. Coding Conventions / コーディング規約

- **Zero Warnings Policy**: All build warnings should be treated as errors. Resolve them immediately (refer to `WarningResolutions.md` in the Vault).
- **ビルド警告ゼロの方針**: すべてのビルド警告はエラーとして扱います。発生した場合は直ちに解決してください（Vault 内の `WarningResolutions.md` を参照）。
- **XML Documentation**: Use 1-line `<summary>` tags for all public/internal members. Use `<remarks>` for additional details.
- **XML ドキュメント**: すべての public/internal メンバーに対して 1 行の `<summary>` タグを使用してください。詳細な情報は `<remarks>` を使用します。
- **Localization**: Never use hardcoded strings in ViewModels. Always use `Strings.xaml` resource dictionaries.
- **ローカライズ**: ViewModel 内で文字列をハードコードしないでください。常に `Strings.xaml` リソース辞書を使用してください。

## 3. Workflow for New Sessions / 新しいセッションのワークフロー

Whenever starting a new task, always run the `/on-boarding` workflow to align with current project state.
新しいタスクを開始する際は、常に `/on-boarding` ワークフローを実行し、現在のプロジェクトの状態に合わせてください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/w-red)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/w-red)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
