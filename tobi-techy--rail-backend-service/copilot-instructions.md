## rail-backend-service

> This repository is a Go backend service. Entry points live in `cmd/`, with `cmd/main.go` as the main server. Core code is under `internal/`: `internal/api` for handlers, middleware, and routes; `internal/domain` for entities and services; `internal/infrastructure` for database, config, and external adapters; and `internal/workers` for background jobs. Shared packages belong in `pkg/`. Migrations live in `migrations/`, config in `configs/`, scripts in `scripts/`, and deployment assets in `deployments/` and `infra/`. Tests live in package-local `*_test.go` files and in `test/` (`test/unit`, `test/integration`, `test/performance`).

# Repository Guidelines

## Project Structure & Module Organization

This repository is a Go backend service. Entry points live in `cmd/`, with `cmd/main.go` as the main server. Core code is under `internal/`: `internal/api` for handlers, middleware, and routes; `internal/domain` for entities and services; `internal/infrastructure` for database, config, and external adapters; and `internal/workers` for background jobs. Shared packages belong in `pkg/`. Migrations live in `migrations/`, config in `configs/`, scripts in `scripts/`, and deployment assets in `deployments/` and `infra/`. Tests live in package-local `*_test.go` files and in `test/` (`test/unit`, `test/integration`, `test/performance`).

## Build, Test, and Development Commands

- `make build` — build `bin/rail_service`.
- `make run` — start the API locally from `cmd/main.go`.
- `make dev` — boot local dependencies with Docker, run migrations, then start the app.
- `make test` — run the full Go test suite with race detection and coverage.
- `make test-coverage` — generate `coverage.out` and `coverage.html`.
- `make lint` — run `golangci-lint`.
- `make security-scan` — run `gosec` and `trivy`.
- `make migrate-up` — apply database migrations.

## Coding Style & Naming Conventions

Target Go `1.24`. Format all Go code with `gofmt`; do not manually tune spacing. Keep package names lowercase, exported identifiers in `CamelCase`, unexported identifiers in `camelCase`, and test files named `*_test.go`. Follow the existing layers: domain logic stays out of handlers, and infrastructure concerns stay out of `internal/domain`. Run `golangci-lint run ./...` before opening a PR.

## Testing Guidelines

Use Go’s `testing` package with `testify` where helpful. Prefer table-driven tests and cover success and failure paths. Run targeted tests during development, for example `go test ./internal/...` or `go test ./test/integration/... -v`. Integration tests expect PostgreSQL and Redis; CI uses Postgres 15 and Redis 7. Keep coverage on changed code meaningful, and update `test/README.md` when you add notable scenarios.

## Commit & Pull Request Guidelines

Recent history follows imperative subjects with prefixes such as `feat:`, `fix:`, `infra:`, `security:`, `perf:`, and `chore:`. Keep commits focused and avoid vague messages like `commit`. PRs should include: a concise summary, linked issue or task, test evidence (`make test` or targeted package tests), and notes for migrations, config changes, or new environment variables. For API changes, include example request/response payloads.

## Security & Configuration Tips

Never commit live secrets or edited `.env` files. Start from `.env.example` and config under `configs/`. Prefer secret managers and environment variables for credentials. If you touch auth, payments, or webhooks, run `make security-scan` before review.

## What we have learned

- Outcome tracking & calibration: `OutcomeTracker` (outcome_tracker.go) records every prediction as a pending outcome, evaluates whether it materialised after the horizon expires, and stores the result in `miriam_prediction_outcomes`. `GetPredictionHitRate` returns accuracy ratio over a lookback period. In `RefreshMoneyState`, the confidence score is multiplied by the hit rate (if > 0) so past prediction accuracy directly scales confidence — a system that's been 60% right can't claim "high confidence". Migration file: `219_miriam_prediction_outcomes`.
- Confidence score is now data-density-aware: tiers based on `activeMonths` for income (0–35) and spending (0–25) signals instead of flat rule-based points. Anomaly detection compares against 6-month trailing average instead of single prior month.
- RampHub UseBread sell (offramp) deposit address is NOT at `providerDetails.depositAddress` or `ourCryptoAddress`. It's nested at `providerDetails.data.deposit.address` (snake_case). Similarly, the send amount is at `providerDetails.data.deposit.amount`. The `ProviderDeposit` struct now has `Address`, `Asset`, `Note` fields for sell-side; `OrderResponse` has `CryptoDepositAddress()` and `CryptoDepositAmount()` helpers to normalize across providers. The sell order normalization in `service.go` calls these before persisting and transferring.
- iMessage bridge uses real `spectrum-ts` SDK (v8.2.1), not the fake `@spectrumlabs/photon` package. Pattern: `Spectrum({projectId, projectSecret, providers:[imessage.config()]})` + `for await (const [space, message] of app.messages)` + `space.send(builder)`. No `.spaces.get()`, no `.on("app.messages")` — real SDK is async-iterator based.
- Account linking is sender-verified: POST `/api/v1/platform/link` issues a one-time token; binding happens when the user texts the token back from iMessage (captures the real platform sender). No insecure `POST /link/confirm` endpoint.
- Action confirmations reuse the orchestrator's existing Redis-backed pending-action store (`ConfirmAction`/`CancelAction`), not a separate HMAC-token store. iMessage has no tap buttons — confirmations use `poll("Confirm: ...", ["Confirm","Cancel"])`. Fund moves (withdrawals/transfers) never use tap-confirm; they require in-app Face ID via `rail://authorize` deep link because messaging can't do passcode step-up.
- ElevenLabs voice: STT via `scribe_v1`, TTS via `eleven_multilingual_v2`. Audio is base64-inlined over RabbitMQ (bridge stays thin, no storage credentials). New REST client at `internal/infrastructure/ai/elevenlabs_rest.go` (existing EL client is WebSocket-only for realtime agent). Gated on `AI_ELEVENLABS_API_KEY` + `AI_ELEVENLABS_VOICE_ID`; degrades to text if unset.
- Persona retuned in `SystemPromptV2` to "cleaner casual": warm/competent, react-first, memory-aware. Roasting is opt-in only. FX bug fixed: `buildNairaContext` injects live rate from `currencyRates.GetLatestRate("USD","NGN")` instead of hardcoded ₦1,600/$1. Voice persona is a separate prompt, not yet aligned with chat persona.
- Supermemory made proactive: `personal_memory` slot in `assembleContext` runs `SearchMemory(userID, message, 6)` for substantive messages (≥12 chars, similarity ≥0.5, 1.2s cap, parallel) so Miriam references past goals/worries/habits without needing an explicit tool call.
- Eval harness: `POST /api/v1/eval/miriam`, gated by `X-Eval-Token`, only registered when `EVAL_ENABLED=true`. Returns content, latency_ms, tokens, provider, pending_action, CheckResponseQuality verdict. Terminal TUI at `cmd/spectrum-bridge/src/eval.ts`: `bun run eval`, slash commands `/user <uuid>`, `/new`, `/whoami`.
- RabbitMQ outbound uses per-platform routing keys (`message.outbound.imessage`) so multiple bridge consumers don't cross-talk. Go consumer declares dead-letter exchange on its queues — permanent failures are dead-lettered, transient (Retryable) errors are requeued. No more poison-loop risk.
- Anomaly Engine at `internal/domain/services/ai/anomaly_engine.go` detects 5 anomaly types: bill_spike, duplicate_charge, fraud_signal, spending_acceleration, merchant_pattern. Each check uses trailing-3-month averages for statistical baselines. Interfaces: `AnomalyCategoryReader`, `AnomalyMerchantReader`, `AnomalyOutflowReader`, `AnomalyFlowReader` — all satisfied by `LedgerSpendingRepository`. Integrated into autopilot morning scan; `BuildAlertText()` converts `[]AnomalyResult` to push notifications. Eval endpoint at `POST /api/v1/eval/anomaly` (gated), TUI command `/anomaly`. 21 unit tests across engine + integration into autopilot.
- Ledger hash chain (migration 283): every `ledger_transactions` row stores `previous_transaction_hash` + its own `transaction_hash=SHA256(prev+fields)`. Tampering with a historical row breaks the chain. `VerifyHashChain(ctx, maxCheck)` detects broken links. Wired into `CheckIntegrity`. Hash computed in `executeTransaction` before insert; first tx has empty previous hash.
- Audit trail on every ledger transaction: `initiated_by` (user/system/admin/webhook/bridge/automation) and `reason` (free-text). `CreateTransactionRequest` requires `InitiatedBy`; defaults to `"system"` in `createTransaction` and `CreatePendingTransaction`. All 15 internal callers set the correct value (TransferStashToSpending=user, AdminTransferStashToSpending=admin, AutomationTransferSpendToStash=automation, etc.).
- Velocity limits (circuit breaker): `ledger_velocity_buckets` table tracks daily outflow + tx count per account. `SetVelocityConfig(cfg)` on `Service` to configure `MaxDailyOutflow` and `MaxDailyTxCount`. Check runs inside `executeTransaction` before each debit entry. Increments bucket atomically after balance update. Only applies to debit (EntryTypeCredit) entries. Nil config = disabled. 31 integration tests pass.
- Circle webhooks: Circle's fungible `/balances` endpoint does NOT include NFTs — NFTs are served by `/v1/w3s/wallets/{id}/nfts` and carry an `nftTokenId` (token id within the contract) distinct from the token UUID. An NFT inbound used to 500-loop because `GetTokenSymbol` only searched `/balances`. `processInboundDeposit` now falls back to `GetNFTTokenID`; resolvable NFTs are returned to sender via `ReturnUnsupportedNFT` (Circle requires `nftTokenIds`, not `tokenId`, for NFT transfers), and unresolvable tokens are acked (200) instead of erroring. Only real API failures or return-transfer failures still surface 500 for retry. Minted spam drops (source = `0x0000...0000` zero address) are never refunded — `isUnreturnableSource` skips empty/zero/burn/same-wallet sources so we don't 500-loop or burn the asset.

---
> Source: [tobi-techy/RAIL-BACKEND-SERVICE](https://github.com/tobi-techy/RAIL-BACKEND-SERVICE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
