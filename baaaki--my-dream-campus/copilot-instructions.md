## my-dream-campus

> Universite yonetim sistemi. Full-stack monorepo: Go moduler monolith (`new-backend/`) + React+Vite web + React Native (Expo) mobil. Eski mikroservis kodu main'den cikarildi — `v0-microservices` git tag'inde arsivli.

# MyDreamCampus — AI Asistanı Talimatları

Universite yonetim sistemi. Full-stack monorepo: Go moduler monolith (`new-backend/`) + React+Vite web + React Native (Expo) mobil. Eski mikroservis kodu main'den cikarildi — `v0-microservices` git tag'inde arsivli.

> Bu dosya **AI'a** talimattir. Kullanici dokumanlari icin bkz. `README.md`.

---

## 1. Cakisma Hiyerarsisi

Cakisma durumunda **yukaridan asagiya** dogru oncelik (1 en yuksek):

1. **Kullanici prompt'u** — en yuksek oncelik
2. **Ilgili `skills.md`** — `new-backend/skills.md`, `frontend/skills.md`, `mobile/skills.md`
3. **Bu dosya (CLAUDE.md)** — proje genel kurallari
4. **Memory** — gecmis konusmalardan sabitler

---

## 2. Zorunlu Okuma Kosullari

Gorev baslamadan **mutlaka oku**:

| Eger suraya dokunacaksan… | Once oku |
|---|---|
| `new-backend/monolith/**` | `new-backend/skills.md` |
| `new-backend/services/**`, `new-backend/shared/**` | `new-backend/skills.md` |
| `frontend/src/**` | `frontend/skills.md` |
| `mobile/app/**`, `mobile/services/**`, `mobile/hooks/**` | `mobile/skills.md` |
| Migration / SQL / sqlc | `new-backend/skills.md` §3-4 (make komutlari + workflow) |

Birden fazla katmanda degisiklik varsa **hepsini** oku.

---

## 3. Konusma Dili & Kod Dili

- **Konusma dili (kullaniciyla)**: Turkce
- **Kod, degisken, dosya adi, commit mesaji**: Ingilizce
- **Kullaniciya hata mesaji (UI text)**: Turkce
- **Log mesaji**: Ingilizce

**YAPMA:** Turkce degisken adi (`kullaniciAdi` ❌, `username` ✅).
**YAPMA:** Ingilizce kullaniciya hata mesaji (`"User not found"` ❌, `"Kullanici bulunamadi"` ✅ — UI'da).

---

## 4. Paket Yoneticisi (kritik — yanlis kullanma)

| Dizin | Komut | YAPMA | Neden |
|---|---|---|---|
| `frontend/` | `bun add`, `bun run`, `bun tsc`, `bunx --bun <x>` | `npm`, `npx`, `yarn` | `bun.lock` source-of-truth; `package-lock.json` yok, npm bagimliliklari farkli cozuyor |
| `mobile/` | `npm install`, `npm run`, `npx expo`, `npx jest` | `bun` (eskiden vardi, kaldirildi) | Expo prebuild scriptleri npm assumption ile yazilmis, `package-lock.json` source |
| `new-backend/monolith/` | `go mod`, `make sqlc-<module>`, `make migrate-up-<module>` (Makefile bu dizinde) | dogrudan `goose`, `sqlc generate` | Makefile `.env` ve modul basina `sqlc.yaml` cozumlemesi yapiyor; ciplak komut config bulamaz |

---

## 5. Docker / sudo Kurali

- Docker komutlari `sudo` gerektiriyor (kullanici `docker` grubunda degil).
- **Sandbox `sudo` calistirmaz** — komutu **kullaniciya kopyala-yapistir** olarak goster, kendin calistirma.

```bash
# Bunu sen calistirma — kullaniciya goster:
sudo docker compose -f new-backend/infrastructure/docker-compose.yml up -d
sudo docker exec mydreamcampus-postgres psql -U postgres -d mydreamcampus -c "SELECT email FROM auth_users;"
sudo docker logs -f mydreamcampus-monolith
```

---

## 6. Onaysiz Yapilabilecekler vs. Sorulacaklar

### SORMA, dogrudan uygula:
- HTTP status code secimi (REST standardi: 200/201/204/400/401/403/404/409/422/500)
- Sifre hash (Argon2id), JWT (HS256) — sabit
- Migration **yazma** (dosya olusturma)
- sqlc query yazma + `make sqlc-<module>` calistirma
- DTO/Repository/Service/Handler iskeleti (skills.md sablonlarini izle)
- Test isimlendirme: `TestXxx_Scenario_ExpectedResult`
- Commit atma (atomic, feature bittiginde)
- Hata mesaji standardi (`platform/errors.AppError`)

### SOR, dogrudan UYGULAMA:
- **Yeni kutuphane** ekleme (ornek: validator icin go-playground vs ozzo)
- **Yeni modul veya servis** olusturma (scope, event semasi, main.go wiring)
- **Yeni event** semasi veya mevcut event payload degisikligi (geriye uyumsuzluk)
- **Migration CALISTIRMA** (`make migrate-up-<module>`) — yazma degil, calistirma sor
- **Sema breaking change** (kolon silme, NOT NULL ekleme, type degisikligi)
- **Frontend route silme** veya yeniden adlandirma
- Onemli refactor (3+ dosya etkileyen, davranis degisikligi)
- `go.mod` / `package.json` dependency guncelleme (patch haric)

---

## 7. Git Commit Formati

```
<type>(<scope>): <description>
```

| Type | Ne zaman |
|---|---|
| `feat` | Yeni ozellik |
| `fix` | Bug fix |
| `chore` | Build, infra, tooling |
| `refactor` | Davranis degismeden yeniden yazim |
| `docs` | Dokumantasyon |
| `test` | Sadece test ekleme |

**Scope:** `auth`, `staff`, `student`, `catalog`, `enrollment`, `attendance`, `grades`, `meal`, `payment`, `notification`, `shared`, `frontend`, `mobile`, `infra`

**Ornekler:**
```
feat(auth): add login and register endpoints
fix(shared): resolve logger initialization bug
chore(infra): update caddy configuration
feat(frontend): add student dashboard page
feat(mobile): implement attendance screen
```

**Kural:** Her ozellik tamamlanınca **HEMEN** commit. Atomic — bir commit bir mantiksal degisiklik.

**YAPMA:**
- `feat: stuff` (scope yok, aciklama yok)
- `update files` (type yok)
- 10 dosya tek commit'te birden cok ozellik
- `--amend` ile push edilmis commit'i degistirme
- `--no-verify` (hook bypass)

---

## 8. Is Bittiginde Checklist (Gorev Kapatmadan Once)

```
Backend feature:
- [ ] Migration yazildi + `make migrate-up-<module>` test edildi (kullanici calistirir)
- [ ] sqlc query yazildi + `make sqlc-<module>` calisti
- [ ] Repository / Service / Handler / DTO yazildi
- [ ] Route modulun `module.go` RegisterRoutes'una baglandi
- [ ] Event publish ediliyorsa outbox'a yaziliyor + consumer'larin (diger modul worker'lari, notification) DTO'lari guncellendi
- [ ] Service-level test yazildi (kritik path'ler — happy + 1 error)
- [ ] `go build ./...` hatasiz
- [ ] Atomic commit atildi

Frontend feature:
- [ ] API service fonksiyonu yazildi (`src/lib/services/`)
- [ ] TanStack Query hook'u var (loading/error state)
- [ ] Type tanimi `src/lib/types.ts`'de
- [ ] Sayfa `src/routes.tsx`'e baglandi
- [ ] `bun tsc --noEmit` hatasiz
- [ ] Tarayicida acilip golden path test edildi
- [ ] Atomic commit atildi

Mobile feature:
- [ ] Service fonksiyonu (`mobile/services/`)
- [ ] Hook (`mobile/hooks/`) — TanStack Query
- [ ] Ekran `mobile/app/` altinda (Expo Router)
- [ ] `npx tsc --noEmit` hatasiz
- [ ] iOS veya Android'de manuel test edildi (loading/error/empty)
- [ ] Atomic commit atildi
```

---

## 9. Failure Mode'lar

### Test basarisiz olursa
**YAPMA:** Test'i `t.Skip()` veya `.skip()` ile atla.
**YAP:** Hatayi oku, fix et veya kullaniciya rapor et. Commit atma.

### Migration basarisiz olursa
**YAPMA:** Tablo manuel `DROP` etme, migration tablosunu `DELETE` etme.
**YAP:** Kullaniciya hatayi goster, `migrate-down` oner.

### Type error (frontend/mobile)
**YAPMA:** `as any`, `@ts-ignore`, `// @ts-expect-error` kullanma.
**YAP:** Type tanimini duzelt. `as unknown as X` dokum gerekiyorsa kullaniciya sor.

### sqlc generate hata verirse
**YAPMA:** Generated dosyalari manuel duzenleme.
**YAP:** Query SQL'ini duzelt, tekrar `make sqlc-<module>`.

### Lint/format hata
**YAP:** Otomatik duzeltilebilenleri duzelt (`gofmt`, `bun tsc`). Logic degisimi gerekirse kullaniciya sor.

---

## 10. Tone & Output

- Cevaplar **kisa** olsun. Diff varsa diff'i konus, kodu tekrar yazma.
- Turkce aciklamalarda **emoji kullanma**.
- Log/yorum yaziminda **NE** degil **NEDEN** acikla. "fetches user" ❌ — "timing-safe: dummy verify against enumeration" ✅.
- Her yerde yorum **ekleme**. Sadece sart oldugunda (gizli kisit, edge case, workaround).
- Kullaniciya rapor verirken: once sonuc, sonra detay. Tersi degil.

---

## 11. Subagent Kullanimi

**Ne zaman kullan:**
- 3+ modulu tarayan arastirma
- Tum kod tabaninda pattern arama
- Karsilastirma analizi (modul A vs modul B'deki yaklasim)

**Ne zaman KULLANMA:**
- Tek dosya okuma
- Bilinen yoldaki dosyayi okuma
- 1-2 grep yeterli olan arama

Kullanici "agent kullan" derse `Agent` tool'u ile `Explore` veya `general-purpose` subagent'i cagir.

---

## 12. Mimari Kararlar (Sabit, Tartisilmaz)

Bu kararlar verilmis — yeniden sorma:

| Konu | Karar |
|---|---|
| Mimari | **Moduler monolith** (`new-backend/monolith`) — tek binary, 9 modul. Notification tek ayri servis (RabbitMQ consumer). |
| Moduller arasi iletisim | **Sync okuma/validasyon:** in-process client interface (HTTP YOK, `X-Internal-Secret` YOK). **Side-effect/notify:** RabbitMQ event + outbox. Client -> backend HTTP, Caddy uzerinden. JWT dogrulamasi `platform/middleware.JWTAuth` ile process icinde. |
| Database | PostgreSQL 18+. Monolith: tek DB, modul basina ayri schema + ayri goose version tablosu. Notification: kendi ayri DB'si (port 5433). |
| ORM/Query | sqlc + pgx/v5 (raw SQL yok, GORM yok) |
| Migration | goose |
| HTTP framework (Go) | Gin v1.11 |
| Auth | JWT HS256 + Argon2id + Redis blacklist |
| Frontend routing | react-router v7 (Next.js YOK) |
| Mobile routing | Expo Router v6 (file-based) |
| Frontend HTTP | ky |
| Mobile HTTP | axios |
| State (web+mobile) | TanStack Query (server state), Context (UI state) |
| Logging | Zap (backend), console (frontend, debug icin) |
| Outbox pattern | Tum event publish'lerde zorunlu |
| Edge / reverse proxy | Caddy (80/443) — `/api/*` -> monolith:8080, geri kalani SPA static (Traefik KALDIRILDI) |

---

## 13. Portlar (referans — `new-backend/infrastructure/docker-compose.yml`)

| Bilesen | Port | Aciklama |
|---|---|---|
| Monolith backend | 8080 | 9 modul tek process icinde. |
| PostgreSQL (monolith) | 5432 | Tek instance, modul basina ayri schema. |
| PostgreSQL (notification) | 5433 | Notification servisinin ayri DB'si. |
| RabbitMQ | 5672/15672 | Event mesajlasmasi + management UI. |
| Redis | 6379 | Token blacklist + rate limit. |
| MailHog | 1025/8025 | Dev SMTP (notification e-postalari). |
| Caddy | 80/443 | Tek public giris: `/api/*` -> monolith, geri kalani SPA. |

Infra portlari `127.0.0.1`'e bind'li — disaridan sadece Caddy erisilir. Frontend dev: `3000` (Vite proxy `/api` -> 8080).

**Compose dosyasi ikiye bolundu** — port publish etme yeri onemli:

| Dosya | Publish ettigi host portlari |
|---|---|
| `docker-compose.yml` (base) | **Sadece** Caddy `:80`. PaaS (Openship) bunu okur; kendi edge'i `:80/:443`'u tuttugu icin baska port publish edilemez. |
| `docker-compose.standalone.yml` | Caddy `:443` + yukaridaki tum infra portlari (`127.0.0.1`). PaaS'siz calisma (LAN / tunnel / VPS) icin. |

`make` hedefleri **ikisini birlikte** yukler — yeni bir host portu eklerken
standalone dosyasina ekle, base'e degil. Detay: [`DEPLOY.md`](DEPLOY.md).

---

## 14. Dokunma — Generated / Korunan Dosyalar

Bu yollardaki dosyalari **manuel duzenleme**. Kaynak dosyayi guncelle ve generator'i tekrar calistir.

| Yol | Kaynak | Regenerate |
|---|---|---|
| `new-backend/monolith/internal/modules/*/db/*.go` | `internal/modules/*/sql/queries/*.sql` | `make sqlc-<module>` |
| `new-backend/**/sql/migrations/*.sql` (uygulanmis) | — | Yeni migration ekle, eskisini degistirme |
| `new-backend/services/notification/internal/db/*.go` | `sql/queries/*.sql` | servis dizininde `sqlc generate` |
| `frontend/src/components/ui/*` | shadcn CLI | `bunx --bun shadcn@latest add <c>` |
| `*.lock`, `*.lockb`, `go.sum`, `bun.lock`, `package-lock.json` | Paket yoneticisi | Komutu calistir, manuel dokunma |

---

## 15. Detayli Rehberler

- Backend: [`new-backend/skills.md`](new-backend/skills.md)
- Frontend: [`frontend/skills.md`](frontend/skills.md)
- Mobile: [`mobile/skills.md`](mobile/skills.md)
- Moduler monolith migration plani (tarihsel referans, migration tamamlandi): `v0-microservices` tag'i altinda `legacy-codebase/architecture/`

## 16. Dis Referanslar

- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [sqlc Docs](https://docs.sqlc.dev/en/latest/)
- [pgx Docs](https://pkg.go.dev/github.com/jackc/pgx/v5)
- [goose Docs](https://pressly.github.io/goose/)
- [Caddy Docs](https://caddyserver.com/docs/)
- [Expo Router Docs](https://docs.expo.dev/router/introduction/)
- [React Router v7 Docs](https://reactrouter.com/)
- [TanStack Query Docs](https://tanstack.com/query/latest)

---
> Source: [Baaaki/my-dream-campus](https://github.com/Baaaki/my-dream-campus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
