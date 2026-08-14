## floor-column-copy

> 作業前に **`docs/project-sync.md`** を読むこと。Launcher 手順は **`docs/bedrock-dev-notes.md`** も参照。

# Floor Column Copy — agent instructions

作業前に **`docs/project-sync.md`** を読むこと。Launcher 手順は **`docs/bedrock-dev-notes.md`** も参照。

## 実装時の必須事項

- **リポジトリ構成・ゲーム仕様を変えたら `docs/project-sync.md` を同期する** — `behavior_packs/`、`config.js`、`ui.js`、`functions/` などを直したら同じ変更で `npm run sync:project-docs` を実行する。
- `npm run sync:project-docs:check` で同期漏れを検出できる（CI 用）。
- スクリプトやテクスチャを直したあとは **ワールド再入場**（できればゲーム再起動）。
- 開発中のワールド反映: `npm run install:bedrock-pack`（初回）→ `npm run sync:bedrock-world-pack` または `npm run dev:bedrock`

## このプロジェクトについて

- Minecraft **Bedrock Edition（統合版）** の Script API アドオン
- 足元から下方向にブロック列をコピー＆貼り付け
- 基本操作は **杖の使用** と `/fc:*`（Beta APIs 不要）。`!fc` チャットのみ Beta APIs が必要

---
> Source: [hakonikomoru/floor-column-copy](https://github.com/hakonikomoru/floor-column-copy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
