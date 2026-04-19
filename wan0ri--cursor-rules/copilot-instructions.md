## cursor-rules

> CI/CD Best Practices for GitHub Actions


# 🚀 CI/CD ルール

- ワークフローは`build`, `test`, `deploy`などに分割してください。
- `jobs`は依存関係を明示し、並列実行を活用してください。
- `secrets`は GitHub Secrets で管理し、直接記述しないでください。
- キャッシュを活用してビルド時間を短縮してください。
- Pull Request 時には自動テストを必ず実行してください。
- デプロイはステージングと本番環境を分けてください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wan0ri) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
