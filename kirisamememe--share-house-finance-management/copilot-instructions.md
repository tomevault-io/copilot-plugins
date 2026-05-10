## share-house-finance-management

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

ルートでの実行（モノレポ）:
```bash
bun install              # 全ワークスペースの依存関係をインストール
bun start                # Backend + Discord Bot を同時起動
bun run backend          # Backend のみ起動（ポート 8008）
bun run discord-bot      # Discord Bot のみ起動
bun run discord-bot:test # Discord Bot のテストのみ実行
bun test                 # 全パッケージのテストを実行（test スクリプトがあるもの）
```

特定パッケージ内で実行する場合:
```bash
cd packages/discord-bot
bun run index.ts         # Bot 起動
bun test                 # テスト実行
bun test src/tests/splitService.test.ts  # 単一テストファイル
```

ローカル Docker（start.sh はルートの .env を読み込む）:
```bash
cp .env.example .env
# .env を編集
bun run docker          # 起動
bun run docker:up       # バックグラウンド起動
```
NAS では start.sh を使わず、コンテナマネージャーで環境変数を手動設定する。

## Monorepo（Bun Workspaces）

`packages/*` がワークスペース。ルートから `bun run --filter <package> <script>` で各パッケージのスクリプトを実行できる。新規パッケージ追加時は `packages/<name>/package.json` を作成し、必要に応じて `workspace:*` で他パッケージに依存。

---

## Architecture

This is a Discord bot for shared house expense tracking (3-person household). It receives receipt images, uses AI to extract transaction data, splits expenses among participants, and requires all 3 members to digitally confirm before recording.

**Current state**: Phase 1 — file-based storage, AI mock, in-memory ledger. Phase 2 will replace mocks with real AI (Vercel AI SDK) and PostgreSQL (Drizzle ORM).

### Request Flow

1. User posts receipt image in `receiptChannelId`
2. `messageCreate` handler downloads image, passes to `aiMock.parse()` → `ExtractedData`
3. `splitService` calculates per-person amounts (see algorithm below)
4. `draftService` writes draft JSON to `data/logs/pending/yyyy-mm-dd_hh-mm-ss-{uuid}.json`
5. Bot posts embed with Confirm/Void buttons
6. As members click Confirm, signatures accumulate in the draft
7. When all 3 sign: draft moves to `committed/`, announcement posted to `financeChannelId`
8. Void button: moves draft to `voided/`, posts cancellation announcement

### State Machine (File-Based)

Directory location is the source of truth — drafts have no `status` field:
- `data/logs/pending/` — awaiting signatures
- `data/logs/committed/` — all signed, ready for DB
- `data/logs/voided/` — cancelled

Concurrent operations on the same draft are serialized via `draftService.withLock(uuid, fn)` (in-memory Promise queue). File moves use `rename()` for atomicity.

### Expense Splitting Algorithm

```
sharedAmount = totalAmount - sum(exclusions)
baseAmount   = floor(sharedAmount / n)
remainder    = sharedAmount % n
```

`remainder` extra yen are distributed to `n` random participants (never the uploader). Each person's final amount = their `baseAmount` + any exclusive items + optional +1 for rounding. Invariant: `sum(finalAmounts) === totalAmount`.

### Key File Responsibilities

（いずれも `packages/discord-bot/` からの相対パス）

| Path | Responsibility |
|------|---------------|
| `src/bot/index.ts` | Client setup, event registration, startup sequence |
| `src/bot/handlers/interactionCreate.ts` | Routes button/command interactions by `custom_id` / `commandName` |
| `src/bot/handlers/draftHandlers.ts` | Confirm (`confirm:{uuid}`) and Void (`void:{uuid}`) logic |
| `src/bot/handlers/depositCommand.ts` | `/deposit` slash command — records deposit, announces to financeChannel |
| `src/bot/services/splitService.ts` | Expense calculation and participant normalization |
| `src/bot/services/draftService.ts` | Draft JSON CRUD, file moves, locking |
| `src/bot/services/embedService.ts` | All Discord embed and button construction |
| `src/bot/startup/registerCommands.ts` | Registers `/deposit` as a Guild slash command at startup |
| `src/bot/mock/aiMock.ts` | Stub for AI parsing — replace in Phase 2 |

### Configuration

**`data/config.json`** (runtime, not committed):
```json
{
  "dashboardMessageId": null,
  "dashboardChannelId": "...",
  "financeChannelId": "...",
  "receiptChannelId": "...",
  "members": { "<discord_user_id>": "Display Name" }
}
```
Exactly 3 members required. See `config.json.example` for template.

**`.env`** (ワークスペースルート、全パッケージ共通):
- `POSTGRES_USER` / `POSTGRES_PASSWORD` / `POSTGRES_DB` / `PG_PORT` — PostgreSQL 接続情報
- `DATABASE_URL_LOCALHOST` — ホストから直接 `bun run db:migrate` するときのみ使用
- `DISCORD_TOKEN` — bot token
- `DISCORD_GUILD_ID` — target server ID for slash command registration
- `GOOGLE_GENERATIVE_AI_API_KEY` — Google Gemini API キー

Bun はワークスペースルートの `.env` を自動ロードするため、全パッケージで共有される。
Docker 実行時は `start.sh --env-file .env` でこのファイルを読み込む。
`.env` は `.dockerignore` により Docker イメージには含まれない。

### Phase 2 Integration Points

`@todo [AI/DB Phase]` markers indicate where mocks are replaced:
- `messageCreate.ts` — `aiMock.parse()` → Vercel AI SDK
- `draftHandlers.ts` — call DB API before `moveToCommitted()`
- `depositCommand.ts` — `ledgerMock.addDeposit()` → POST `/ledger/deposit` API
- `src/bot/mock/ledgerMock.ts` — entire file replaced by PostgreSQL

### Access Control Policy

This bot operates in a **private Discord server accessible only to the 3 household members**. Third parties cannot reach the bot's channels under any normal circumstance.

- **No permission error messages**: If an unknown user ID (not in `config.members`) somehow triggers an interaction, silently ignore it — no replies, no error feedback of any kind.
- **No access-denied UX**: Unknown actors are simply dropped.
- Implementation: unknown-user interactions fall through guards (`if (!draft) return`, `requiredSigners.includes()`, etc.) and produce no side effects.

Do **not** add permission error handling or any "unauthorized" response anywhere in the codebase.

### Discord.js Constraints

- Slash commands: call `deferReply()` within 3 seconds. Tokens are always fresh (no 10062 risk).
- Buttons (confirm/void): use `message.edit()` to update embeds — does NOT require an interaction token. `tryDeferUpdate()` sends ACK but 10062 is silently ignored; the edit proceeds regardless.
- Do NOT use `ephemeral` / `MessageFlags.Ephemeral` anywhere in this codebase.
- Interaction tokens expire after 15 minutes.

---
> Source: [Kirisamememe/share-house-finance-management](https://github.com/Kirisamememe/share-house-finance-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
