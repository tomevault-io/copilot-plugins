## cubiqport

> Cubiqport proje mimarisi, tech stack ve genel kurallar


# Cubiqport — Proje Mimarisi

## Stack
- **Monorepo**: npm workspaces → `apps/api`, `apps/web`, `packages/shared`
- **API**: Fastify + Drizzle ORM + PostgreSQL (Neon)
- **Web**: Next.js 14 App Router (`(dashboard)` layout grubu)
- **Shared**: `@cubiqport/shared` — tipler ve Zod şemaları
- **Deployment**: `deploy.sh` → SSH ile sunucuya → PM2 + Nginx

## Katman Kuralları
Her modül **4 katmandan** oluşur; sorumluluklar karışmaz:
```
routes.ts     → sadece Fastify route kaydı + JSON schema
controller.ts → HTTP parse + response, iş mantığı yok
service.ts    → iş mantığı, SSH, DB sorguları
repository.ts → sadece Drizzle/DB erişimi (gerekirse)
```

## Paylaşılan Tipler
- Tüm API ↔ Web arası tipler `packages/shared/src/types/index.ts` içinde
- Yeni tip eklemeden önce shared'e bak, var mı kontrol et

## SSH İşlemleri
- `buildSshOptions(server)` kullan → `apps/api/src/utils/ssh-credentials.ts`
- Her SSH metodu `NodeSSH` bağlantısını `finally` bloğunda `dispose()` eder

## Streaming / Uzun İşlemler
- Uzun SSH işlemleri **SSE job pattern** ile yapılır → `tech-jobs.ts`
- `createJob → markRunning → appendLog → finishJob` sırasına uy
- Frontend SSE'yi `EventSource` + `onmessage` ile tüketir

## Analiz İşleri (Test / Analiz)
- **Domain analizleri** (SEO, stress test, security scan): API sürecinde **inline** çalışır (kuyruk yok).
- PM2: `cubiqport-worker` isteğe bağlı; analiz işleri API’de çalıştığı için boşta kalır.
- Yeni analiz tipi: `run-analysis-job.ts` + `addAnalysisJob` (queue modülü inline tetikler).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahmetdaldemir) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
