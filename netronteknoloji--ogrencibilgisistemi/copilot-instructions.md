## ogrencibilgisistemi

> Multi-tenant okul yönetim sistemi. Web paneli + REST API + MAUI mobil. Her okul ayrı SQL DB. ZKTeco kart okuyucu entegrasyonu.

# CLAUDE.md - OgrenciBilgiSistemi

Multi-tenant okul yönetim sistemi. Web paneli + REST API + MAUI mobil. Her okul ayrı SQL DB. ZKTeco kart okuyucu entegrasyonu.

## Çözüm (6 proje)

```
OgrenciBilgiSistemi/         → MVC paneli (EF Core, ZKTeco, SignalR)
OgrenciBilgiSistemi.Api/     → REST API (JWT, ham SQL)
OgrenciBilgiSistemi.Mobil/   → MAUI (Android + iOS)
OgrenciBilgiSistemi.Shared/  → Ortak modeller (TenantBaglami, OkulBilgiAyari)
OgrenciBilgiSistemi.Sms/     → SMS (ilksms.com)
OgrenciBilgiSistemi.Push/    → FCM push bildirimleri
```

Her projenin **kendi CLAUDE.md**'si vardır — detaylar oradadır.

## Bağımlılık Grafiği

```
MVC, Api  → Shared, Sms
Mobil     → Shared (HTTP üzerinden Api'ye)
```

**MVC ↔ API arasında doğrudan referans YOK.** Aynı DB'yi paylaşırlar.

## MVC vs API

| | MVC | API |
|---|---|---|
| Auth | Cookie | JWT Bearer |
| Tenant | Cookie/Session | JWT `okulKodu` claim |
| DB | EF Core | Raw `SqlClient` |
| Authorize | Global FallbackPolicy | Manuel `[Authorize]` |
| Background svc | 5 | 1 |

## Multi-Tenancy

- Tenant listesi: `appsettings.json` → `Okullar[]`.
- DbContext **per-request connection string** alır.
- Migration sadece MVC'de; deploy için idempotent script üretip her DB'de çalıştır.

## Genel Yasaklar

- ❌ MVC ↔ API `ProjectReference` ile bağlama.
- ❌ API'de EF Core, MVC'de raw SQL.
- ❌ `DbContextPool` / `AddDbContextPool`.
- ❌ Tenant context olmadan DB sorgusu.
- ❌ ZKTeco çağrısını MVC dışından yapma.
- ❌ MVC için Any CPU / x64 build (ZKTeco COM 32-bit → **x86 zorunlu**).
- ❌ Soft delete'li entity'leri (`RandevuModel`, `OgretmenRandevuModel`, `BildirimModel`, `DuyuruModel`) `Remove()` ile silme → `IsDeleted = true`.
- ❌ Hassas veri (TC, şifre, kart UID, JWT secret, telefon, SMS içeriği) loglama veya commit.
- ❌ JWT secret'ı versiyona giren config'e yazma → env variable.
- ❌ Mobil'de hassas veriyi `Preferences`'a koyma → `SecureStorage`.

## Konfigürasyon

`Okullar[]`, `SmsAyarlari` ve `GenelAdmin` MVC + API'de aynı (üçü de duplicated, her iki backend de okur).

| Sadece | Anahtar |
|---|---|
| API | `Jwt`, `Cors:AllowedOrigins`, `MvcWwwRootPath` |
| Mobil | `KayitSunucuUrl` (embedded resource) |

## Komutlar

```bash
# Çalıştır
dotnet run --project OgrenciBilgiSistemi/OgrenciBilgiSistemi.csproj
dotnet run --project OgrenciBilgiSistemi.Api/OgrenciBilgiSistemi.Api.csproj
dotnet build OgrenciBilgiSistemi.Mobil -t:Run -f net10.0-android

# Migration (sadece MVC)
dotnet ef migrations add <Isim> --project OgrenciBilgiSistemi/OgrenciBilgiSistemi.csproj
dotnet ef database update --project OgrenciBilgiSistemi/OgrenciBilgiSistemi.csproj
dotnet ef migrations script --idempotent --project OgrenciBilgiSistemi/OgrenciBilgiSistemi.csproj
```

## Naming

- Entity: `*Model` · Service: `I*Service`/`*Service` · View: `*View` · Hub/Filter/Middleware: aynı suffix
- DbSet/tablo: çoğul Türkçe (`Ogrenciler`)
- Kod dili: Türkçe (sınıf, method, namespace dahil)

## Dağıtım

- MVC + API → tek IIS, **x86 app pool**. API, MVC'nin `wwwroot/uploads`'unu serve eder.
- Mobil → Google Play / App Store, API'ye HTTPS.
- Her okul için ayrı SQL Server DB.

## Önemli Davranışlar

- **`AppDbContext.IncludeDeleted`** flag'i çoğu entity'de global query filter'ı kontrol eder. Pasif öğrenci raporu için `true` yap.
- **Profil tabloları** (`VeliProfilModel`, `ServisProfilModel`, `OgretmenProfilModel`): PK = `KullaniciId`.
- **Menu seed**: 34 menü (Id 1-34), yeni eklerken Id 35'ten başla.
- **SMS retry** her iki backend'de de bağımsız hosted service olarak var.

## İlk Bakış Sırası

1. `ANALIZ_RAPORU.md`
2. `OgrenciBilgiSistemi/Data/AppDbContext.cs`
3. Her iki `Program.cs`
4. İlgili projenin `CLAUDE.md`'si

---
> Source: [netronteknoloji/OgrenciBilgiSistemi](https://github.com/netronteknoloji/OgrenciBilgiSistemi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
