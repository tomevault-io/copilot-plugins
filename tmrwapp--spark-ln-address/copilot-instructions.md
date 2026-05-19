## spark-ln-address

> Lightning Address provider implementing LNURL-Pay (LUD-06/LUD-16) and LNURL-Auth (LUD-04).

# spark-ln-address

Lightning Address provider implementing LNURL-Pay (LUD-06/LUD-16) and LNURL-Auth (LUD-04).
BOLT11 invoices are created via `@buildonspark/spark-sdk`; users self-register usernames by
signing a k1 challenge with their Spark identity key.

## Stack

NestJS 10 / TypeScript 5.1 (CommonJS, ES2021, loose tsconfig: `strictNullChecks: false`,
`noImplicitAny: false`) · Prisma 6 + MySQL 8 · `@noble/secp256k1` 3 · `bech32` 2 · Jest 29.
Prettier: no semicolons, single quotes, trailing commas all. Files: kebab-case.

## Setup & Commands

```
npm install && docker-compose up -d mysql && npm run start:dev
```

Scripts: `build`, `start[:dev|:debug|:prod]`, `lint`, `format`, `test[:watch|:cov|:debug]`,
`test:e2e[:setup|:teardown|:full]`. App listens on **3003**. Dev MySQL **3308**, test MySQL **3309**.

## Endpoints

- `GET /.well-known/lnurlp/:username` — LNURL-Pay metadata
- `GET /lnurl/callback/:username?amount=<msat>` — returns BOLT11
- `GET /v1/auth/lnurl` — issues k1 challenge
- `GET /v1/auth/lnurl/callback?k1&sig&key&username` — verifies, creates `User` + `LightningName`
- `GET /v1/query/username/:pubKey`
- `GET /v1/query/pubkey/:username`

## File Map

```
src/
  main.ts, app.module.ts
  config/                NestJS ConfigModule (loads .env.test → .env.local → .env)
  lnurl/                 LNURL-Pay controller/service
  auth/                  LNURL-Auth (k1 issuance + signature verification)
  query/                 username <-> pubkey lookups
  lightspark/            SparkWallet wrapper (creates invoices)
  common/                utils.ts (normalizeUsername), constants.ts, spark-address.utils.ts
  prisma/                PrismaService
prisma/schema.prisma     User, LightningName, AuthNonce, Invoice
test/jest-e2e.setup.ts   mocks Spark SDK for E2E
docker-compose.yml       mysql (3308), mysql-test (3309, --profile test)
```

## Domain Gotchas

- **msat bounds:** `MIN_SENDABLE_MSAT = 1_000`, `MAX_SENDABLE_MSAT = 10_000_000_000`
  (`src/common/constants.ts`). Convert to sats with `Math.floor(msat / 1000)`.
- **Username normalization:** `normalizeUsername` enforces `^[a-z0-9._-]{1,30}$`
  (`src/common/utils.ts`). Always normalize before DB lookup.
- **k1 dual-path verification** (`src/auth/auth.service.ts`): k1 is 64 hex chars. The verifier
  tries the signature against both the raw 32 bytes (standard LNURL) AND the UTF-8 bytes of the
  hex string (Spark's `signMessageWithIdentityKey`). Both DER and compact sigs, both compressed
  and uncompressed pubkeys, are accepted. Do not simplify this.
- **Dynamic secp256k1 import:** `@noble/secp256k1` v3 is ESM-only. Loaded via
  `new Function('specifier', 'return import(specifier)')` to survive the CommonJS build —
  do NOT replace with a static `import` or TS will downlevel it to `require`.
- **SparkWallet init is fire-and-forget** (`src/lightspark/lightspark.service.ts`):
  `SparkWallet.initialize(...)` is not awaited in the constructor; `createInvoice` throws
  `'Spark wallet not initialized'` if called before the promise resolves.
- **`amountMsat` is `BigInt`** in Prisma — serialize accordingly.
- **`AuthNonce`:** 5-minute expiry, single-use (`usedAt`). `Invoice` 24h expiry.
  Cascade deletes from `User`.
- **`queryByUsername` uses `contains`,** not exact match — by design but easy to miss
  (`src/query/query.service.ts`).
- **Spark address bech32m encoding** (`src/common/spark-address.utils.ts`): HRPs
  `spark`/`sparkt`/`sparkrt`/`sparks`/`sparkl`; payload `0x0A 0x21` + 33-byte compressed pubkey.
  Encoding errors are swallowed in the query service and returned as `null`.

## Conventions

- Throw `BadRequestException` / `NotFoundException` from controllers/services.
- `private readonly logger = new Logger(ClassName.name)` in every service.
- LNURL-Auth error responses must be `{ status: 'ERROR', reason: string }` per spec.

## Env

Required: `DATABASE_URL`, `PUBLIC_BASE_URL`. Optional: `PORT` (3003), `NODE_ENV`,
`SPARK_NETWORK` (default `MAINNET`). Example in `env.example`. E2E uses
`mysql://spark_user:spark_password@localhost:3309/spark_ln_address_test`.

---
> Source: [tmrwapp/spark-ln-address](https://github.com/tmrwapp/spark-ln-address) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
