## emi-core

> - **Composer**: `lbhurtado/emi-core`

# emi-core — AI Agent Context

## Package Identity
- **Composer**: `lbhurtado/emi-core`
- **Namespace**: `LBHurtado\EmiCore`
- **Purpose**: Provider-agnostic EMI domain — contracts, enums, models, events
- **Laravel**: 12.x or 13.x
- **Pattern**: Laravel Actions (`lorisleiva/laravel-actions`), Spatie Laravel Data DTOs

## Directory Layout
```
src/
  Contracts/     — 6 interfaces (WalletProvider, TransferProvider, CashInProvider, CashOutProvider, SignsProviderPayloads, VerifiesProviderPostbacks)
  Enums/         — 8 enums (ProviderCode, WalletType, WalletStatus, ComplianceLevel, VerificationStatus, TransactionType, TransactionStatus, TransactionDirection)
  Models/        — 12 Eloquent models with enum casts and relationships
  Actions/       — (reserved for orchestration actions)
  Data/          — (reserved for Spatie Data DTOs)
  Events/        — (reserved for domain events)
  Exceptions/    — (reserved)
  Support/       — (reserved)
  ValueObjects/  — (reserved)
config/          — emi-core.php
database/
  migrations/    — 8 migration files creating: provider_accounts, wallets, wallet_profiles, wallet_limit_snapshots, transactions, transfers, bank_accounts, cash_ins, cash_outs, otp_challenges, webhook_receipts, reconciliation_entries
  factories/     — (reserved for model factories)
tests/
  TestCase.php   — extends Orchestra\Testbench\TestCase
  Pest.php       — pest()->extend(TestCase::class)->in(__DIR__)
  Unit/Enums/    — enum case/value tests
  Feature/Models/ — model create/cast/relationship tests
  Feature/Package/ — service provider boot, config, migration smoke tests
```

## Model Relationships
- `ProviderAccount` hasMany `Wallet`
- `Wallet` hasOne `WalletProfile`, hasMany `WalletLimitSnapshot`, hasMany `BankAccount`
- `Transaction` belongsTo `Wallet` (wallet_id, source_wallet_id, destination_wallet_id)
- `Transfer` belongsTo `Transaction`
- `CashIn` belongsTo `Transaction`
- `CashOut` belongsTo `Transaction`, belongsTo `BankAccount`, hasOne `OtpChallenge`
- `ReconciliationEntry` belongsTo `Transaction`

## Key Conventions
- All transactions use `request_id` (unique, indexed) as the primary correlation key
- Enum casts on all status/type fields — never store raw strings without an enum
- Models use `$fillable` arrays (not `$guarded`)
- JSON `meta` columns on most models for provider-specific overflow data
- Migrations use `foreignId()->constrained()` with appropriate cascade behavior

## Testing
- Run via host: `composer test:emi-core`
- TestCase bootstraps in-memory SQLite, runs all package migrations
- Factory guessing: `LBHurtado\EmiCore\Database\Factories\{Model}Factory`

## What This Package Does NOT Do
- No HTTP calls — that's the adapter package's job
- No provider-specific logic — all provider logic lives in adapter packages
- No controller/route definitions — the adapter or host app provides those

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/3neti)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/3neti)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
