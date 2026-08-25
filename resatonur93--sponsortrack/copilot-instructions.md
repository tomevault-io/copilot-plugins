## sponsortrack

> Next.js 14 (App Router) · TypeScript · Prisma 5 + PostgreSQL · NextAuth v4 · Tailwind CSS + Radix UI · Supabase Storage / AWS S3 · Nodemailer

# SponsorTrack — Codebase Guide

## Tech Stack

Next.js 14 (App Router) · TypeScript · Prisma 5 + PostgreSQL · NextAuth v4 · Tailwind CSS + Radix UI · Supabase Storage / AWS S3 · Nodemailer

## Commands

```bash
npm run dev          # geliştirme sunucusu
npm test             # vitest birim testleri
npm run test:watch   # izleme modunda test
npm run test:coverage
npm run db:migrate   # prisma migrate dev
npm run db:seed      # test verisi
npm run db:studio    # Prisma Studio
```

## Architecture

### Multi-tenancy

Her şirket bir `Tenant` kaydıdır. `lib/prisma.ts` içindeki `prisma` (tenant-scoped) client, tüm sorgulara otomatik `tenantId` filtresi ekler. Cron ve admin işlemleri için `prismaBase` (filtresiz) kullanılır.

### Two Compliance Systems — Ne Zaman Hangisi?

Kodda iki paralel uyum takip sistemi vardır. İkisi kasıtlı olarak ayrı tutulmuştur:

| Sistem | Model | Enum | Ne için? |
|--------|-------|------|----------|
| **Bildirim sistemi** | `NotificationEvent` | `NotificationType` | Otomatik tetiklenen, tarih-tabanlı hatırlatıcılar (vize süresi, RTW, belge bitişi). Cron tarafından oluşturulur, idempotency key ile tekrar korunur. AO onayı beklenmez. |
| **Uyum olayları** | `ComplianceEvent` | `EventType` | Manuel olarak kaydedilen, Home Office'e bildirilmesi gereken olaylar (no-show, maaş değişikliği, görev değişikliği). HR → Manager → AO → REPORTED iş akışı vardır. |

**Kritik kural:** Aynı olayı **her iki sisteme birden yazmayın.**

- Bir iş eylemi `ComplianceEvent` üretir → HO raporlama iş akışına girer.
- Bir tarih penceresi dolduğunda `NotificationEvent` üretilir → inbox bildirimi olur, iş akışı yoktur.

Örtüşen EventType/NotificationType değerleri (örn. `WORK_LOCATION_CHANGE`, `SALARY_REDUCTION`):
- `ComplianceEvent.EventType.WORK_LOCATION_CHANGE` → HR tarafından manuel oluşturulan olay
- `NotificationType.WORK_LOCATION_CHANGE` → sistemin otomatik ürettiği bildirim

Yeni bir raporlanabilir olay eklerken hangisine yazacağınıza bu ayrıma göre karar verin.

### Risk Göstergeleri

Üç ayrı risk değeri vardır; bunlar farklı amaçlar için tasarlanmıştır:

| Gösterge | Kaynak | Ne için? |
|----------|--------|----------|
| `Worker.complianceRiskLevel` | Worker kaydedilirken/güncellenirken hesaplanan hafif snapshot | Worker listesinde hızlı filtreleme |
| `computeRiskSnapshot()` — `app/api/workers/[id]/route.ts` | Worker GET sırasında inline hesaplama (notification + doc) | Worker detay sayfasında anlık gösterim |
| `RiskScore` modeli | `lib/risk-scoring-engine.ts` + cron | Dashboard risk raporu, trend takibi |

`RiskScore` motoru en kapsamlı olandır. Üç gösterge arasında çelişki varsa `RiskScore` motoruna güvenin.

### Cron Mimarisi

`/api/cron/run` → `lib/scheduler/index.ts` → `runDailyCron()` + `runAlertsPipeline()`

Worker döngüleri `lib/scheduler/batch-iterate.ts::forEachInBatches` ile 100'lük batch'ler halinde işlenir. Tüm worker'ları belleğe almaz.

Bireysel cron endpoint'leri (`/api/cron/escalation`, `/api/cron/risk-scores`, `/api/cron/process-alerts`) de mevcuttur ancak `/api/cron/run` hepsini zaten orkestrasyonla çağırır. Dış zamanlayıcıyı yalnızca `/api/cron/run`'a bağlayın.

### Document Vault vs. Compliance Documents

| Model | Amaç |
|-------|------|
| `DocumentVault` | Ham dosya deposu — S3/Supabase'e yüklenen her dosya buraya kaydedilir. `DocumentFolder` enum ile klasörlenir. |
| `Document` | Uyum kaydı — belirli bir uyum olayına (veya bağımsız olarak) bağlı resmi belge kaydı. `DocumentVaultFolder` enum ile klasörlenir. |

`Document.vaultFileId` → `DocumentVault.id` ilişkisi ile bir vault dosyası en fazla bir uyum belgesine bağlanabilir.

Tüm klasörler `DocumentFolder` tek enum'unda tanımlıdır (`ROLE_DUTIES`, `ROLE_ORG_CHART`, `OTHER` dahil). `DocumentVaultFolder` enum'u kaldırıldı — yeni klasör eklerken yalnızca `DocumentFolder`'ı güncelleyin.

### Güvenlik Katmanları

1. **IP whitelist** (`TenantSecuritySettings.enforceIpWhitelist`) — Login sırasında `lib/security/tenant-login-ip.ts` kontrol eder
2. **Session guard** (`lib/security/session-guard.ts`) — Her API isteğinde `authSid` JWT'si DB'deki `UserAuthSession` ile doğrulanır; idle/absolute timeout uygulanır
3. **Tenant scope** — `prisma` client tüm sorgulara otomatik `tenantId` filtresi ekler

SYSTEM_ADMIN IP whitelist'ten muaftır.

## Testing

```
tests/
  lib/
    dates.test.ts              # tarih yardımcıları
    deadline-rules.test.ts     # HO deadline hesaplamaları
    notification-rules.test.ts # idempotency key üretimi, pencere hesaplamaları
    salary-record-utils.test.ts # pro-rata maaş hesabı, CSV parse
    risk-scoring-engine.test.ts # risk faktörü hesabı, seviye eşikleri
```

DB bağımlılığı olan kodlar (scheduler, API route'ları) için test yazarken `prismaBase`'i mock'layın.

---
> Source: [resatonur93/sponsortrack](https://github.com/resatonur93/sponsortrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
