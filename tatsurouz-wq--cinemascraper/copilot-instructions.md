## cinemascraper

> CineSyncは関東圏の映画館スケジュールを横断比較するWebアプリケーション。

# CineSync Project Rules

## プロジェクト概要
CineSyncは関東圏の映画館スケジュールを横断比較するWebアプリケーション。
monorepo構成で、Pythonスクレイパー（scraper/）とNext.jsフロントエンド（web/）で構成。

## 技術スタック
- Backend: Python 3.12 + uv + Playwright
- Frontend: Next.js + React + Tailwind CSS + pnpm
- Hosting: GitHub Pages (static export)
- CI/CD: GitHub Actions

## コーディング規約
- Python: ruff でフォーマット・リント。型ヒント必須。Pydantic v2使用
- TypeScript: strict mode。ESLint + Prettier
- テスト: pytest (Python) / Jest or Vitest (TypeScript)
- コミット: Conventional Commits形式

## ディレクトリ構成
- `scraper/` - Python スクレイパー（uv管理）
- `web/` - Next.js フロントエンド（pnpm管理）
- `data/schedules/` - スクレイパー出力JSON（自動更新）
- `.github/workflows/` - CI/CD定義

## 重要な制約
- スクレイピング間隔: 最低2秒
- robots.txt を必ず確認
- GitHub Pages: 静的エクスポートのみ（next.config.ts の output: 'export'）
- データはJSONファイルとしてリポジトリに格納（DBなし）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tatsurouz-wq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tatsurouz-wq)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
