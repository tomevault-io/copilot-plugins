## cashier-mollie-dotnet

> ASP.NET Core 10 Library fuer Mollie Subscription-Management — Feature-Paritaet mit `mollie/laravel-cashier-mollie`.

# CashierMollie for .NET

ASP.NET Core 10 Library fuer Mollie Subscription-Management — Feature-Paritaet mit `mollie/laravel-cashier-mollie`.

## Zweck

Open-Source NuGet-Package das Mollie Recurring Payments in ASP.NET Core integriert,
analog zu Laravel Cashier Mollie. Wird von ModCockpit als erstes Produkt genutzt (Dogfooding).

## Stack

| Komponente | Technologie |
|-----------|-------------|
| Framework | ASP.NET Core 10, .NET 10 |
| ORM | Entity Framework Core 10 |
| Payment API | Mollie.Api 4.18.0 |
| Tests | xUnit, NSubstitute, EF Core InMemory |
| Code Quality | TreatWarningsAsErrors, AnalysisLevel latest-recommended |

## Projektstruktur

```
cashier-mollie-dotnet/
├── CashierMollie.sln
├── .editorconfig                — C# Code-Style, Naming-Konventionen
├── .github/workflows/ci.yml    — GitHub Actions CI (build, test, NuGet publish)
├── CHANGELOG.md                 — Versionierte Aenderungshistorie
├── CONTRIBUTING.md              — OSS Contribution Guidelines
├── Directory.Build.props        — TreatWarningsAsErrors, EnforceCodeStyleInBuild
├── src/CashierMollie/
│   ├── CashierMollie.csproj     — Version 0.3.0
│   ├── CashierMollieOptions.cs  — Konfiguration (ApiKey, Currency, BillingEngine, etc.)
│   ├── Models/              — Subscription, OrderItem, Payment, Order, Credit, Refund,
│   │                          RedeemedCoupon, Coupon, SubscriptionResult, SwapOptions,
│   │                          SubscriptionOptions, PaymentMethodUpdateResult, ChargeResult,
│   │                          IHasTimestamps, IHasConcurrencyToken
│   ├── Interfaces/          — IBillable, ICashierService, ISubscriptionBuilder, IChargeBuilder,
│   │                          IBillingEngine, IMollieClientService, IWebhookService,
│   │                          ICashierEventDispatcher, ICouponRepository, ICouponHandler,
│   │                          ICouponService, ICreditService, IRefundService, IInvoiceGenerator
│   ├── Services/            — CashierService, SubscriptionBuilder, ChargeBuilder,
│   │                          MollieBillingEngine, ManagedBillingEngine,
│   │                          CashierBackgroundService, MollieClientService,
│   │                          WebhookService, CouponService, ConfigCouponRepository,
│   │                          FixedDiscountHandler, PercentageDiscountHandler,
│   │                          CreditService, RefundService, NullInvoiceGenerator,
│   │                          NullCashierEventDispatcher
│   ├── Extensions/          — ServiceCollectionExtensions, ApplicationBuilderExtensions,
│   │                          ModelBuilderExtensions
│   ├── Middleware/           — CashierWebhookMiddleware
│   ├── Exceptions/          — CashierException
│   └── Events/              — 23 domain events (CashierEvents.cs)
└── tests/CashierMollie.Tests/
    ├── CashierMollie.Tests.csproj
    ├── Models/              — SubscriptionTests
    ├── Services/            — CashierServiceTests, MollieBillingEngineTests,
    │                          ManagedBillingEngineTests, CashierBackgroundServiceTests,
    │                          MollieClientServiceTests, WebhookServiceTests,
    │                          CouponServiceTests, CreditServiceTests, RefundServiceTests,
    │                          ChargeBuilderTests, NullCashierEventDispatcherTests
    ├── Integration/         — SubscriptionLifecycleTests, CouponIntegrationTests,
    │                          CreditIntegrationTests, RefundIntegrationTests,
    │                          ManagedEngineIntegrationTests
    └── TestHelpers/         — TestDbContextFactory, TestBillable
```

## Implementierter Feature-Stand (v0.3.0)

### Subscription-Lifecycle
- Create, Cancel, CancelImmediately, Resume, Swap, Grace Periods, Trial
- Quantity Management (Update, Increment, Decrement)
- Multiple Subscriptions per Owner

### Dual Billing Engine (Strategy Pattern)
- **MollieBillingEngine** (default) — delegiert an Mollie Subscription API
- **ManagedBillingEngine** — eigener Billing-Cycle mit OrderItem-Scheduling + Mandate Payments
- **CashierBackgroundService** — Timer-basiert, ruft ProcessDueItemsAsync auf (nur Managed)
- Konfigurierbar via `BillingEngineType` Enum (MollieNative / Managed)

### Payment & Mandate
- First Payment Flow (Checkout-Redirect fuer Mandate-Erstellung)
- Payment Method Update (Mandate-Erneuerung)
- Mandate Validierung + Revoke
- One-off Charges (IChargeBuilder, mit/ohne Mandate)

### Coupon System
- ICouponRepository (default: ConfigCouponRepository aus appsettings)
- ICouponHandler (FixedDiscountHandler, PercentageDiscountHandler)
- Validierung, Einloesung, Widerruf

### Credit / Balance System
- Per-Owner, per-Currency Credit-Balance
- AddCredit, ApplyCredit (gedeckelt auf Balance), GetBalance, HasCredit

### Refund System
- Partial + Complete Refunds via Mollie API
- Lokales Refund-Tracking mit AmountRefunded auf Payment

### Chargeback Detection
- Automatische Erkennung via Webhook (AmountChargedBack)
- ChargebackReceived Event

### Webhook-Handling
- Opt-in Middleware (`UseCashierWebhook()`)
- FirstPaymentPaid/Failed Events fuer Pending Subscriptions
- Subscription-Aktivierung nach erfolgreicher Erstzahlung

### Invoice Interface
- IInvoiceGenerator<TKey> (default: NullInvoiceGenerator)
- Consumer ersetzt mit eigener Implementierung (sevdesk, PDF, etc.)

### Events
- 23 Domain Events via ICashierEventDispatcher
- Subscription, Payment, Mandate, Coupon, Credit, Refund, Chargeback, Order Events

### DI Registration
- `AddCashierMollie<TKey>()` registriert alle Services
- Automatische Engine-Auswahl + BackgroundService basierend auf Config
- TryAddScoped fuer ersetzbare Defaults (EventDispatcher, CouponRepository, InvoiceGenerator)

## DB-Tabellen (7)

cashier_subscriptions, cashier_order_items, cashier_payments,
cashier_orders, cashier_credits, cashier_refunds, cashier_redeemed_coupons

## Build & Test

```bash
dotnet build   # 0 Warnings, 0 Errors (TreatWarningsAsErrors aktiv)
dotnet test    # 283 Tests
```

## Konventionen

- **Sprache Code:** Englisch
- **Sprache Docs:** Deutsch (TheOlymp-Kontext)
- **Naming:** C# Standard (PascalCase Public, _camelCase Private Fields)
- **Async:** Alle Mollie-API-Calls async (Task<T>)
- **Nullable:** Aktiviert (nullable reference types)
- **Code Style:** .editorconfig (Braces when_multiline, var only when type apparent)
- **Analyzer:** AnalysisLevel latest-recommended, CA1707/CA1816/CA1859 in Tests unterdrueckt
- **LoggerMessage:** Source-generierte Delegates (CA1848 compliant)

## Bekannte Einschraenkungen

- **~95% Feature-Paritaet** mit Laravel Cashier Mollie
- Fehlend: Proration-Berechnung, Built-in Invoice-Renderer, Plan-Preis-Resolution, Dunning/Retry
- GitHub: https://github.com/theolymp/cashier-mollie-dotnet

## Referenz

- Laravel Cashier Mollie: https://github.com/laravel/cashier-mollie
- Mollie API Docs: https://docs.mollie.com/
- Mollie.Api NuGet: https://www.nuget.org/packages/Mollie.Api

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theolymp)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/theolymp)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
