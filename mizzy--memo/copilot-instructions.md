## memo

> Obsidianの代替として開発する個人用メモツールWebアプリケーション。

# Memo App

Obsidianの代替として開発する個人用メモツールWebアプリケーション。
ObsidianのMac/iPhone間の同期不安定と、iOSアプリの起動の遅さが主な移行動機。

## Tech Stack

| Layer | Tech |
|---|---|
| API | Hono + TypeScript |
| Frontend | React + Vite + TypeScript |
| Editor | Tiptap (ProseMirrorベース) |
| UI | Tailwind CSS + shadcn/ui |
| DB | Cloudflare D1 + Drizzle ORM |
| Storage | Cloudflare R2 (画像添付) |
| Hosting | Cloudflare Workers (単一Workerで静的アセット+APIを配信) |
| Auth | Cloudflare Access (Cloudflareアカウントでログイン, 個人Gmailのみ許可) |
| Monorepo | Turborepo |
| Infra | Terraform (Cloudflare provider), stateはR2に保存 |

## Project Structure(予定)

```
memo/
├── apps/
│   ├── api/          # Hono (Cloudflare Workers、webのdistも配信)
│   └── web/          # React + Vite (SPA)
├── packages/
│   └── shared/       # 型定義の共有
├── infra/            # Terraform (Cloudflare Access, D1, R2 etc.)
├── package.json      # monorepo root
└── turbo.json
```

## Initial Scope

- メモの作成・編集・一覧・検索
- Vault機能(常に1つのVaultを開き、Vaultスイッチャーで切り替える)
- フォルダ機能(Vault内に任意深さのフォルダ木を持ち、3ペイン構成でフォルダツリー/メモ一覧/エディタを表示する)
- PWA対応(iPhoneからホーム画面に追加して利用)

## Design Decisions

- SPA構成 (SSR不要、個人ツールなのでSEO不要)
- 単一Worker構成: 1つのWorkerが静的アセット(SPA)とAPIを配信。同一オリジンでCORS不要、Cloudflare Accessも1ホスト分で済む (当初のPages+Workers分離案から変更)
- Hono RPCでAPIとフロントエンド間の型安全な通信
- Cloudflare Accessでアプリ前段の認証 (アプリ内認証コード不要)
- Terraform stateはCloudflare R2に保存 (S3互換バックエンド)
- R2のstate用バケットはwrangler CLIで手動作成 (bootstrap)

---
> Source: [mizzy/memo](https://github.com/mizzy/memo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
