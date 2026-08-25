## eraberu

> - **サービス**: 住宅営業マンのマッチングサービス。¥1,000で詳細プロフィールを開示する

# 営業マンの通知表サイト〜ERABERU〜 - CLAUDE.md

## プロジェクト概要

- **サービス**: 住宅営業マンのマッチングサービス。¥1,000で詳細プロフィールを開示する
- **URL**: https://eigyo-no-tsuchihyo.vercel.app
- **スタック**: Next.js 16 + Supabase + Tailwind CSS v4 + TypeScript
- **デプロイ**: Vercel（git pushで自動デプロイ）
- **Supabase**: プロジェクト `sales-review`（ID: `jydawtmlshofviszztbu`）

### プロダクトファミリー（SETUP Lab）
| 略称 | サービス名 | 内容 |
|---|---|---|
| ERABERU | 営業マンの通知表サイト | 営業マンを選ぶ（★現在開発中） |
| MANABERU | 営業の大学 | 営業マンが学ぶ |
| MITUKERU | 土地の直売所 | 土地を見つける |
| HANASERU | 理想のお宅見学 | 施主同士が話せる |
| EGAKERU | 住宅要望整理 | 理想の家を描く |

---

## 開発ルール

- `'use client'`のみ使用。Server Componentは使わない
- Supabaseクライアントは `createBrowserClient` のみ（`src/lib/supabase.ts`）
- 環境変数: `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_ANON_KEY`（Vercelにも設定済み）
- 環境変数を変更したら必ずVercelでRedeploy（NEXT_PUBLIC_*はビルド時に焼き込まれる）

---

## Supabase テーブル構成

| 名前 | 種別 | 用途 |
|------|------|------|
| `salesperson_profiles` | TABLE | 営業マンの本体データ（実名・bio等を含む） |
| `safe_salesperson_profiles` | VIEW | 公開用（個人情報をマスク済み） |
| `unlocked_profiles` | TABLE | 決済済み開示レコード（buyer_id, agent_id） |
| `profiles` | TABLE | ユーザープロフィール |
| `contract_reviews` | TABLE | 成約後レビュー |

### RLS現状
- `salesperson_profiles`: anon → 安全列のみGRANT（real_name等は不可）。authenticated → `unlocked_profiles`に記録がある行のみ取得可能
- `unlocked_profiles`: authenticated → 自分の行（buyer_id = auth.uid()）のみ取得可能
- `safe_salesperson_profiles`: anon/authenticated ともに GRANT SELECT 済み

---

## 決済フロー（Stripe）

1. フロントエンド → Edge Function `create-checkout-session` を呼び出し
2. Stripe Checkout（テストカード: `4242 4242 4242 4242`）
3. Webhook `stripe-webhook-eraberu` → `unlocked_profiles` にレコード挿入
4. 詳細ページ再読み込みで実名等が表示される

---

## トラブルシューティング

- **カードが表示されない**: `safe_salesperson_profiles`はViewのためRLSはベーステーブルに設定する
- **ビルドエラー**: `supabase.ts`は`createBrowserClient`のみ使用、`'use client'`を忘れずに
- **Vercelで動かない**: 環境変数を確認してRedeploy

---

## 実装TODO

→ [TODO.md](./TODO.md) を参照

---
> Source: [reimagininghomelab-boop/ERABERU](https://github.com/reimagininghomelab-boop/ERABERU) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
