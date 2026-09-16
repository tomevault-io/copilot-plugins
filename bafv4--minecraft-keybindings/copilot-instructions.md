## minecraft-keybindings

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Minecraft Keybindings is a Next.js application for sharing and viewing Minecraft Java Edition keybinding configurations. Users authenticate via Discord OAuth, then manually set their Minecraft profile (MCID and UUID) and can create/edit their keybinding settings which are stored in a PostgreSQL database.

## Development Commands

### Setup & Installation
```bash
pnpm install
pnpm prisma generate
pnpm prisma db push
```

### Development
```bash
pnpm dev          # Start dev server at http://localhost:3000
pnpm build        # Build production bundle
pnpm start        # Start production server
pnpm lint         # Run ESLint
```

### Database Operations
```bash
pnpm prisma generate    # Generate Prisma Client after schema changes
pnpm prisma db push     # Push schema changes to database (development)
pnpm prisma studio      # Open Prisma Studio GUI
```

### Device Data Management
```bash
pnpm fetch-devices      # Fetch gaming mice and keyboards from Rakuten API
```

## Environment Variables

Required in `.env.local`:
- `DATABASE_URL` - PostgreSQL connection string (Neon recommended)
- `NEXTAUTH_URL` - Application URL (http://localhost:3000 for dev)
- `NEXTAUTH_SECRET` - Generated via `openssl rand -base64 32`
- `DISCORD_CLIENT_ID` - Discord app client ID
- `DISCORD_CLIENT_SECRET` - Discord app client secret
- `CRON_SECRET` - Secret for Vercel Cron job authentication (generated via `openssl rand -base64 32`)
- `RAKUTEN_APP_ID` - Rakuten API Application ID (optional, for device data fetching)

## Architecture

### Authentication Flow

The auth system ([lib/auth.ts](lib/auth.ts)) supports two authentication methods:

**1. MCID/Password Authentication (Credentials)**
- Users register with MCID, optional UUID, and password
- Password is hashed using bcryptjs (10 salt rounds)
- Direct login with MCID and password

**2. Discord OAuth (Optional)**
- User logs in with Discord account via NextAuth.js
- After first login, users are automatically redirected to MCID setup page
- Middleware ([middleware.ts](middleware.ts)) enforces MCID setup before accessing other pages

Note: Automatic Minecraft profile retrieval via Xbox Live API is not implemented due to Microsoft approval requirements.

### Data Model

The database schema follows NextAuth.js adapter requirements with additional Minecraft-specific fields:

**Account** (NextAuth model)
- OAuth provider accounts linked to users
- Stores provider tokens and metadata

**Session** (NextAuth model)
- User session tokens
- Links to User via userId

**User** (Prisma model in [prisma/schema.prisma](prisma/schema.prisma))
- Primary fields: `id`, `mcid` (username, unique), `uuid` (player ID, unique), `password` (hashed)
- Discord OAuth fields: `name`, `email`, `emailVerified`, `image` (NextAuth adapter requirements)
- Timestamps: `createdAt`, `updatedAt`
- Relationships: one-to-one with PlayerSettings, one-to-many with Account and Session

**PlayerSettings** (Prisma model)
- Stores complete keybinding configuration with defaults matching vanilla Minecraft
- Mouse settings: DPI, game sensitivity, Windows speed, acceleration, cm/180
- Keybindings: movement, actions, inventory, hotbar (stored as Minecraft key format: `key.keyboard.w`, `key.mouse.left`)
- JSON fields for advanced config:
  - `remappings` - Hardware-level key remaps (e.g., Caps Lock → Ctrl)
  - `externalTools` - AutoHotKey/macro configurations
  - `additionalSettings` - Future extensibility

### Key Format Convention

All keybindings use Minecraft's internal key naming:
- Keyboard: `key.keyboard.w`, `key.keyboard.left.shift`, `key.keyboard.space`
- Mouse: `key.mouse.left`, `key.mouse.right`, `key.mouse.middle`

This format is used throughout the schema defaults and should be preserved when adding new keybinding fields.

### API Routes

アプリケーションは17個のRESTful APIエンドポイントを提供しています。以下は主要なエンドポイントの概要です：

**プレイヤー情報取得（公開API）**
- `GET /api/player/[mcid]` - 個別プレイヤーの完全な情報を取得
- `GET /api/players` - 設定を持つ全プレイヤーのリストを取得

**認証・登録**
- `POST /api/auth/register` - 新規ユーザー登録（Mojang API連携）
- `POST /api/auth/login-or-register` - ログインまたは自動登録
- `GET /api/auth/check-mcid` - MCID存在チェック

**設定管理（要認証）**
- `GET /api/keybindings` - 認証済みユーザーの設定を取得
- `POST /api/keybindings` - プレイヤー設定を更新（管理者はゲストユーザーも編集可能）
- `DELETE /api/keybindings` - 全設定を削除

**アイテム配置**
- `GET /api/item-layouts` - アイテム配置を取得
- `POST /api/item-layouts` - アイテム配置を作成・更新
- `DELETE /api/item-layouts` - アイテム配置を削除

**ゲスト管理（管理者のみ）**
- `GET /api/guests` - ゲストユーザー一覧
- `POST /api/guests` - ゲストユーザー作成
- `DELETE /api/guests` - ゲストユーザー削除

**MCID同期**
- `GET /api/sync-mcid?uuid={uuid}` - 個別ユーザーのMCIDを同期
- `POST /api/sync-mcid` - 全ユーザーのMCIDを同期（Cron用）

**アバター**
- `GET /api/avatar?uuid={uuid}&size={size}` - Mojang APIからアバター画像を生成

### API Documentation

完全なAPI仕様書（17エンドポイント、リクエスト/レスポンス例、エラーハンドリング、使用例を含む）：
- **マークダウン形式**: [docs/API_SPECIFICATION.md](docs/API_SPECIFICATION.md)
- **OpenAPI形式**: [docs/openapi.yaml](docs/openapi.yaml)

### Session Management

NextAuth session is extended via TypeScript declaration merging ([types/next-auth.d.ts](types/next-auth.d.ts)) to include Minecraft-specific fields (`mcid`, `uuid`) populated by the session callback in [lib/auth.ts](lib/auth.ts).

### Database Client

Prisma client ([lib/db.ts](lib/db.ts)) uses singleton pattern with global caching to prevent multiple instances during development hot-reloading.

### MCID Synchronization

Since Minecraft users can change their MCID (username) while keeping the same UUID, the application implements automatic MCID synchronization:

**Automatic Daily Sync**
- Vercel Cron job runs daily at midnight (configured in [vercel.json](vercel.json))
- Fetches latest MCID for all users via Mojang Session Server API
- Updates database records when MCID changes are detected
- Rate-limited to respect Mojang API limits (100ms delay between requests)

**Manual Sync**
- Users can click the "同期" (Sync) button on the edit page
- Immediately fetches and updates their current MCID from Mojang API
- Displays confirmation message showing old and new MCID if changed

**Implementation Details**
- Uses Mojang Session Server API: `https://sessionserver.mojang.com/session/minecraft/profile/{uuid}`
- UUID is the primary key and never changes
- MCID updates preserve all user settings and relationships

## Component Architecture

**Player Display Components**
- `MinecraftAvatar` - Renders Minecraft player head from skin API
- `PlayerCard` - Shows player overview with avatar and basic info
- `KeybindingDisplay` - Read-only keybinding visualization
- `KeybindingEditor` - Edit form for keybinding configuration

**Pages**
- `/` - Homepage with player list
- `/player/[mcid]` - Player profile with keybindings
- `/player/[mcid]/edit` - Edit page (protected, requires matching session)
- `/login` - Discord OAuth login page

## Discord OAuth Configuration

### Overview

このアプリケーションは Discord OAuth で認証を行います。ログイン後、ユーザーは手動で Minecraft アカウント情報（MCID と UUID）を設定する必要があります。

### 詳細な設定手順

#### 1. Discord Developer Portal でアプリケーション作成

1. [Discord Developer Portal](https://discord.com/developers/applications) にアクセス
2. **New Application** ボタンをクリック
3. アプリケーション名を入力（例: "Minecraft Keybindings"）
4. 利用規約に同意して **Create** をクリック

#### 2. OAuth2 設定

1. 左メニューから **OAuth2** → **General** を選択
2. **Client ID** をコピー → `.env.local` の `DISCORD_CLIENT_ID` に設定
3. **Client Secret** の **Reset Secret** をクリック
4. 表示される Secret をコピー → `.env.local` の `DISCORD_CLIENT_SECRET` に設定
   - ⚠️ この値は一度しか表示されないため、必ず保存してください

#### 3. Redirect URIs の設定

1. **Redirects** セクションで **Add Redirect** をクリック
2. 開発環境用 URI を追加: `http://localhost:3000/api/auth/callback/discord`
3. **Save Changes** をクリック

#### 4. 本番環境用の Redirect URI の追加

Vercel などにデプロイする際:
1. **OAuth2** → **General** の **Redirects** セクション
2. 本番環境の URI を追加: `https://your-domain.vercel.app/api/auth/callback/discord`
3. **Save Changes** をクリック

### 環境変数の設定例

```env
# Database
DATABASE_URL="postgresql://user:password@host/database?sslmode=require"

# NextAuth設定
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-generated-secret-here"

# Discord OAuth
DISCORD_CLIENT_ID="1234567890123456789"
DISCORD_CLIENT_SECRET="your-discord-client-secret-here"
```

### トラブルシューティング

**"redirect_uri_mismatch" エラー**
- Discord Developer Portal で設定した Redirect URI と実際のコールバック URL が一致していない
- 正確に `http://localhost:3000/api/auth/callback/discord` が設定されているか確認

**ログインできない / 環境変数エラー**
- `.env.local` の環境変数が正しく設定されているか確認
- Discord の Client ID と Client Secret が正しいか確認
- 開発サーバーを再起動（環境変数変更後は必須）

## Type System

Type definitions in [types/player.ts](types/player.ts) mirror Prisma schema but use TypeScript interfaces for frontend consumption. Key types:
- `MouseSettings` - Mouse configuration subset
- `Keybinding` - All keybinding fields
- `RemappingConfig` - Key remapping dictionary
- `ExternalToolsConfig` - External tool actions by tool name
- `PlayerSettings` - Complete settings (extends Keybinding & MouseSettings)

## Deployment (Vercel)

When deploying to Vercel:
1. Set all environment variables in Vercel dashboard
2. Update `NEXTAUTH_URL` to production URL
3. Add production redirect URI to Discord app: `https://your-domain.vercel.app/api/auth/callback/discord`

{
  "permissions": {
    "allow": [
      "Bash(npx create-next-app@latest minecraft-keybindings --typescript --tailwind --app --no-src --import-alias \"@/*\")",
      "Bash(npx create-next-app@latest minecraft-keybindings --typescript --tailwind --eslint --app --no-src --import-alias \"@/*\")",
      "Bash(npm install:*)",
      "Bash(npx tailwindcss:*)",
      "Bash(pnpm install:*)",
      "Bash(pnpm add:*)",
      "Bash(nul)",
      "Bash(cat:*)",
      "Bash(pnpm tsc:*)",
      "Bash(pnpm remove:*)",
      "Bash(openssl rand:*)",
      "Bash(pnpm prisma generate:*)",
      "Bash(pnpm dev)",
      "Bash(pnpm prisma db push:*)",
      "Bash(npx @auth/prisma-adapter)",
      "Bash(taskkill:*)",
      "Bash(dir:*)",
      "Bash(copy .env.local .env)",
      "Bash(timeout:*)",
      "Bash(set)",
      "Bash(unset:*)",
      "Bash(del /F \".next\\dev\\lock\" 2)",
      "Bash(if exist \".next\\dev\\lock\" del /F \".next\\dev\\lock\")",
      "Bash(PRISMA_USER_CONSENT_FOR_DANGEROUS_AI_ACTION=\"yes\" pnpm prisma db push:*)",
      "Bash(powershell -Command:*)",
      "Bash(set FORCE_COLOR=1)",
      "Bash(ping:*)",
      "Bash(set PRISMA_USER_CONSENT_FOR_DANGEROUS_AI_ACTION=\"yes\")"
    ],
    "deny": [],
    "ask": []
  }
}

---
> Source: [bafv4/minecraft-keybindings](https://github.com/bafv4/minecraft-keybindings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
