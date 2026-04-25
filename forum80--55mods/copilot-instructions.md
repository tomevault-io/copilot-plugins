## 55mods

> Oyun modu paylaşım platformu. Kullanıcılar mod yükler, indirir, yorum yapar; creator'lar stüdyo üzerinden içerik yönetir; adminler moderasyon ve site yönetimi yapar.

# 55Mods – Project Brain (CLAUDE.md)

Oyun modu paylaşım platformu. Kullanıcılar mod yükler, indirir, yorum yapar; creator'lar stüdyo üzerinden içerik yönetir; adminler moderasyon ve site yönetimi yapar.

---

## Tech Stack

### Backend
- **PHP 8.2+** — Custom DDD mimarisi (Laravel DEĞİL)
- **MySQL 8.0** — InnoDB, utf8mb4, veritabanı adı: `55mods`
- **JWT** — `firebase/php-jwt` ile access + refresh token sistemi
- **PSR-7** — `nyholm/psr7` HTTP mesaj katmanı
- **vlucas/phpdotenv** — Ortam değişkenleri

### Frontend
- **Vue 3.3** — Composition API, `<script setup>` syntax
- **TypeScript 5.3**
- **Vite 5** — Build tool, output: `../public`
- **Pinia 2** — State management
- **Vue Router 4** — 95+ route
- **Tailwind CSS 3.3** — Ana CSS framework (dinamik tema sistemi var)
- **Headless UI** — Erişilebilir unstyled bileşenler
- **Heroicons** — İkon sistemi (FontAwesome KULLANMA, kaldırılıyor)
- **Axios** — API iletişimi, 30s timeout, otomatik token refresh
- **Chart.js** — Admin dashboard grafikleri
- **DOMPurify** — HTML sanitizasyonu
- **vuedraggable** — Drag & drop
- **vite-plugin-pwa** — PWA + Service Worker

### Geliştirme Ortamı
- **Docker Compose** — MySQL + PHP/Apache + phpMyAdmin
  - MySQL: `localhost:3306`
  - PHP/Apache: `localhost:8080`
  - phpMyAdmin: `localhost:8081`
- **Vite Dev Server** — `localhost:5173` (frontend)
- **Context7 MCP** — Güncel kütüphane dokümantasyonu

---

## Geliştirme Ortamı Kurulumu

### Docker (MySQL + PHP Backend)
```bash
# 1. Docker Desktop kur: https://www.docker.com/products/docker-desktop/
# 2. Servisleri başlat
docker compose up -d

# 3. XAMPP'tan DB'yi taşı
bash scripts/db-export.sh
# Çıktıdaki komutu çalıştır

# 4. .env güncelle (DB_HOST=mysql, docker için)
```

### Frontend (Vite)
```bash
cd frontend
npm install
npm run dev   # localhost:5173
```

### XAMPP ile çalışmaya devam (geçiş dönemi)
```
DB_HOST=localhost
DB_PORT=3306
APP_URL=http://localhost/55Mods
```

---

## Proje Yol Haritası

### PHASE 0: Altyapı & Araçlar
| Görev | Durum | Notlar |
|-------|-------|--------|
| CLAUDE.md oluşturuldu | ✅ Tamamlandı | |
| Context7 MCP kuruldu | ✅ Tamamlandı | ~/.claude/settings.json |
| Docker Compose hazırlandı | ✅ Tamamlandı | 3 servis: mysql, php, phpmyadmin |
| .env güvenlik temizliği | ✅ Tamamlandı | SSH bilgileri kaldırıldı |
| .env.example güncellendi | ✅ Tamamlandı | |
| DB export scripti | ✅ Tamamlandı | scripts/db-export.sh |
| Docker Desktop kurulumu | ✅ Tamamlandı | v29.3.1 |
| XAMPP→Docker DB geçişi | ✅ Tamamlandı | 3.1MB, tüm tablolar aktarıldı |

### PHASE 1: i18n Sistemi Yenileme ✅ Tamamlandı (2026-03-28)
| Görev | Durum | Notlar |
|-------|-------|--------|
| vue-i18n v9 kurulumu | ✅ Tamamlandı | `npm install vue-i18n@9` |
| Mevcut JSON dosyaları dönüştürme | ✅ Tamamlandı | public/languages/ — format zaten uyumlu, taşıma gerekmedi |
| Custom store/composable kaldırma | ✅ Tamamlandı | 485 satır custom impl → vue-i18n wrapper (~100 satır) |
| main.ts: plugin + initI18n() | ✅ Tamamlandı | `useLocaleStore` kaldırıldı, `initI18n()` kullanılıyor |
| utils/dateHelper.ts güncelleme | ✅ Tamamlandı | `useLocaleStore()` → `i18n.global` |
| TypeScript tip güvenliği | ✅ Tamamlandı | i18n kaynaklı 0 hata, build başarılı |
| Tolgee kurulumu (opsiyonel) | ⬜ Bekliyor | Görsel çeviri editörü |

#### Phase 1 Mimari Kararları
- `src/plugins/i18n.ts` — vue-i18n instance, `loadLocale()`, `setLocale()`, `initI18n()`
- `src/composables/useI18n.ts` — ince wrapper; eski API birebir korundu (t, te, tm, locale, setLocale, formatNumber, formatDate, formatRelativeTime)
- `src/stores/i18n.ts` — geriye-uyumluluk re-export'u, silinmedi
- JSON dosyaları `public/languages/` — dinamik fetch, bundle'a dahil değil
- Fallback: EN → TR sırasıyla yüklenir

### PHASE 2: Kalite Kontrolü — Frontend ✅ Tamamlandı (2026-03-28)
| # | Parça | Durum | Notlar |
|---|-------|-------|--------|
| 2.1 | Auth & Güvenlik | ✅ Tamamlandı | JWT, guards, token refresh — solid, değişiklik gerekmedi |
| 2.2 | Tema Sistemi | ✅ Tamamlandı | Tailwind CSS vars + API entegreli theme store — solid |
| 2.3 | Frontend Mimari | ✅ Tamamlandı | 0 TS hatası: User.stats, ApiResponse.meta, img callable, DataWidgets, WeeklyChoice, vb. |
| 2.4 | Mod Sistemi | ✅ Tamamlandı | fetchComments → api.mods.getComments(); pagination meta fix |
| 2.5 | WIP Sistemi | ✅ Tamamlandı | Backend routes doğrulandı, 0 TS hatası |
| 2.6 | Pack Sistemi | ✅ Tamamlandı | hasMoreComments pagination fix; 0 TS hatası |
| 2.7 | Oyun/Kategori/Marka | ✅ Tamamlandı | 0 TS hatası, solid |
| 2.8 | Kullanıcı & Sosyal | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.9 | Creator Studio | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.10 | Admin Paneli | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.11 | Arama & Keşif | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.12 | Chat & Mesajlaşma | ✅ Tamamlandı | 0 TS hatası — solid |
| 2.13 | Bildirimler | ✅ Tamamlandı | 0 TS hatası — solid |

#### Phase 2.3 Yapılanlar (2026-03-28)
- `types/index.ts`: User.stats, User.points, User.stats.total_mods/packs/wips, ApiResponse.meta, ModRequest.comment_count, ModCollection.collection_type genişletildi, ModCollection.tags, WeeklyWinner.entity_title/category/nominee
- `services/api.ts`: modRequests.getComments, addComment, claim, close, complete() overload eklendi; ApiResponse.meta eklendi
- `utils/imageHelper.ts`: `img` callable yapıldı (img(path, 'mods') çalışır, artı img.mod() gibi method form korundu)
- `components/widgets/renders/DataWidgets.ts`: interval → cleanupInterval düzeltildi
- `views/admin/UserDetail.vue`: 'unverify' action type eklendi
- `views/admin/Reports.vue`: reported_entity.slug eklendi
- `components/admin/settings/ServicesTab.vue`: false vs string boolean karşılaştırması düzeltildi
- `views/series/*Premium.vue`, `views/requests/*Premium.vue`, `views/WeeklyChoice.vue`: possibly undefined fixes

### PHASE 3: Kalite Kontrolü — Backend ✅ Tamamlandı (2026-03-28)
| # | Parça | Durum | Notlar |
|---|-------|-------|--------|
| 3.1 | DDD Yapısı | ✅ Tamamlandı | 80+ entity, 38 repo arayüzü, value objects, domain events — solid |
| 3.2 | API Endpoints | ✅ Tamamlandı | Route sıralaması doğru (static önce, parameterized sonra) |
| 3.3 | Güvenlik | ✅ Tamamlandı | SQL injection fix, CORS env dokümantasyonu, rate limiting aktif |
| 3.4 | Performans | ✅ Tamamlandı | Kapsamlı index'ler, N+1 yok (loadAllRelations sadece detail'de), CacheMiddleware aktif |

#### Phase 3 Yapılanlar (2026-03-28)
- **SQL Injection Fix**: `AbstractMysqlRepository::findWithFilters()` → `getAllowedSortColumns()` allowlist + `getAllowedSortColumns()` abstract method eklendi
- **MysqlModRepository** → `getAllowedSortColumns()` override: title, download_count, view_count, rating_average, is_featured, published_at
- **MysqlPackRepository** → `getAllowedSortColumns()` override: title, mod_count, download_count, view_count
- **MysqlWIPRepository** → `getAllowedSortColumns()` override: title, progress, last_activity_at
- **ModController** → sortMap dışı `$sortBy` değerlerine `else { $sortBy = 'created_at'; }` fallback eklendi
- **Route Ordering Bug Fix**: `config/api.php` — `GET /packs/my` static route `GET /packs/{id}` parameterized route'un ÖNÜNE taşındı
- **Güvenlik Durumu**: CORS env-configurable ✅, RateLimitMiddleware aktif ✅, HtmlSanitizer DOMDocument-tabanlı ✅, FileUploadService extension+MIME+archive integrity ✅, ImageUploadService getimagesize() ✅, TokenAuthService DB hash tokens ✅
- **DDD Bulguları**: MysqlUserRepository AbstractMysqlRepository'yi extend etmiyor (teknik borç), servis arayüzleri eksik, `getRecentWithAuthors()` dead code + N+1 (çağrılmıyor, kritik değil)

### PHASE 4: İkon Sistemi Standardizasyonu
| Görev | Durum | Notlar |
|-------|-------|--------|
| FontAwesome kullanımlarını bul | ⬜ Bekliyor | fas/far/fab sınıfları |
| Heroicons karşılıklarına geç | ⬜ Bekliyor | |

### PHASE 5: Animasyon & Polish
| Görev | Durum | Notlar |
|-------|-------|--------|
| @vueuse/motion kurulumu | ⬜ Bekliyor | Sayfa geçişleri |
| Skeleton loaders tamamlama | ⬜ Bekliyor | |
| Micro-interactions | ⬜ Bekliyor | Button, form, card |

---

## Mimari

### Backend — DDD Katmanları
```
src/
  Domain/{Feature}/
    Entity/           # Domain modelleri
    Repository/       # Repository arayüzleri
    ValueObject/      # Değer nesneleri
  Application/{Feature}/
    Service/          # Business logic
    DTO/              # Data transfer objects
  Infrastructure/{Feature}/
    Repository/       # Repository implementasyonları
  Presentation/
    Controller/{Feature}/  # HTTP controller'lar
    Http/
      Routing/        # Custom router
      Middleware/     # Auth, RateLimit middleware
```

### Frontend — Feature-Based
```
frontend/src/
  views/        # admin/, creator/, games/, mods/, auth/, profile/...
  components/   # admin/, cards/, mods/, chat/, ui/, widgets/
  stores/       # Pinia store'ları
  composables/  # Vue composable'ları
  services/     # api.ts — tüm API çağrıları
  types/        # index.ts — TypeScript tipleri
  config/       # paths.ts, site.ts
```

---

## Konvansiyonlar

| Tür | Kural | Örnek |
|-----|-------|-------|
| Vue bileşeni | PascalCase | `ModCardPro.vue` |
| Composable | camelCase + `use` | `useI18n.ts` |
| Pinia store | camelCase | `auth.ts` |
| PHP sınıfı | PascalCase | `UserController` |
| Migration | snake_case tarihli | `2026_01_20_add_game_image_positions.php` |

### Bileşen Varyantları
- `Premium` suffix = gelişmiş versiyon: `ModListPremium.vue`
- `Pro` suffix = kart bileşeni: `ModCardPro.vue`
- Router'a bak, hangisi kullanıldığını oradan anla

### API Çağrıları
- **Sadece `frontend/src/services/api.ts` üzerinden**
- 401 → otomatik token refresh
- Response → `normalizeApiResponse()` ile normalize

---

## Önemli Komutlar

### Docker
```bash
docker compose up -d          # Servisleri başlat
docker compose down           # Durdur
docker compose logs -f php    # PHP logları
docker exec -it 55mods_mysql mysql -u root -psecret 55mods
```

### Frontend
```bash
cd frontend
npm run dev          # localhost:5173
npm run build        # Production build → ../public
npm run type-check   # TypeScript
npm run lint         # ESLint + fix
npm run validate     # Hepsini çalıştır
```

### Backend
```bash
php artisan                                    # Komut listesi
php database/migrate.php migrate               # Migration
php database/migrate.php rollback [n]          # Geri al
php database/migrate.php fresh                 # Sıfırla
bash scripts/db-export.sh                      # DB yedekle
```

---

## State Management

| Store | Ne İşe Yarar |
|-------|--------------|
| `auth.ts` | User, token, login/logout |
| `settings.ts` | Site ayarları (API'dan) |
| `theme.ts` | Tema, CSS değişkenleri |
| `i18n.ts` | Çoklu dil → vue-i18n'e geçecek |
| `messaging.ts` | Chat/mesajlaşma |
| `toast.ts` | Bildirimler |

**Auth storage:** `localStorage` → `access_token`, `refresh_token`

---

## Router Guard'ları
```typescript
meta.requiresAuth    // → login
meta.requiresAdmin   // → /forbidden
meta.requiresCreator // → /forbidden
meta.guest           // → authenticated ise home
```

---

## Önemli Dosyalar

| Dosya | Ne İşe Yarar |
|-------|--------------|
| `bootstrap.php` | DI container, middleware, routing |
| `config/api.php` | 100+ API endpoint (route config) |
| `frontend/src/services/api.ts` | Tüm API fonksiyonları |
| `frontend/src/router/index.ts` | Tüm route'lar |
| `frontend/src/types/index.ts` | TypeScript tipleri |
| `frontend/tailwind.config.js` | Tema sistemi, 630 satır |
| `public/.htaccess` | Apache routing + güvenlik |
| `docker-compose.yml` | Geliştirme ortamı |

---

## Bilinen Sorunlar & Teknik Borç

### Yarım Kalmış
- Chat: tarih ayırıcı, reaction picker, reply scroll, typing indicator, team oluşturma
- Bazı view'larda `// TODO:` yorumları

### Tutarsızlıklar
- **İkon:** Heroicons (doğru) + FontAwesome kalıntıları (PHASE 4'te kaldırılacak)
- **Dual bileşenler:** `ModList.vue` vs `ModListPremium.vue` — router'a bak
- **API field normalizasyonu:** `avatar` vs `avatar_url` → `apiNormalizer.ts` hallediyor
- **Hardcoded Türkçe:** PHASE 1'de vue-i18n ile çözülecek

### Dikkat
- `config/api.php`: static route'lar parameterized'dan ÖNCE gelmeli
- `main.ts`: store init için 3s timeout (Promise.race)
- Rate limiting MySQL'de (Redis değil) — performans kısıtı

---

## Yapılmaması Gerekenler
- FontAwesome (`fas`, `far`, `fab`) kullanma → Heroicons
- `any` TypeScript tipi kullanma → strict typing
- API çağrısını direkt axios ile yapma → `services/api.ts`
- Metin hardcode etme → i18n kullan
- SSH/şifre bilgisi `.env`'e yazma → `.env.production` kullan (gitignore'da)
- Route sıralamasını bozma (config/api.php)

---

## Güvenlik Notları
- `.env` gitignore'da — git'e gitmez ✅
- `.env.example` şifre içermez ✅
- Deployment bilgileri `.env.production`'a taşındı ✅
- CORS şu an `*` → production'da kısıtlanmalı ⚠️
- JWT_SECRET mutlaka değiştirilmeli ⚠️

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/forum80) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
