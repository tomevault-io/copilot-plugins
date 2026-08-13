## portfoy-takip

> Bu dosya Cursor ve benzeri araçlar için **proje özgü** kısa kurallar içerir. Genel kodlama tercihleri kullanıcı ayarlarında kalır.

# Agent / geliştirici notları (portfoy-takip)

Bu dosya Cursor ve benzeri araçlar için **proje özgü** kısa kurallar içerir. Genel kodlama tercihleri kullanıcı ayarlarında kalır.

## Kalite ve çözüm yaklaşımı (zorunlu)

- **Quick & dirty yok:** Geçici, acele veya “şimdilik çalışsın” diye bırakılan çözümler yazma. Kök nedeni anla; mevcut mimari, `lib/` ve hook’larla **tutarlı**, sürdürülebilir çözüm üret.
- **Yama (patch) yok:** Belirtiyi gizleyen, tekrarlayan mantık, kopyala-yapıştır workaround, gereksiz `if`/`try-catch` fallback’leri veya “bir yere daha ekle” tarzı düzeltmeler yapma. Sorun ortak katmanda çözülmeli.
- **Yama şartsa onay al:** Gerçekten geçici bir yama veya teknik borç kaçınılmazsa **uygulamadan önce** kullanıcıya açıkça yaz: ne yapılacak, neden kalıcı çözüm şimdi mümkün değil, sonraki adım ne. **Onay olmadan yama commit etme.**
- **Önbellek / semptom düzeltmesi:** Dosya doğru olsa bile “cache temizle”, “reload”, “flag ekle” gibi semptom müdahaleleri kalıcı çözüm yerine geçmesin; gerekirse onayla ve borcu not et.

## Stack
- **Expo SDK 54** + **expo-router**, **Supabase** (auth + Postgres + RLS), **EAS** ile iOS/Android build.
- Ortam: `.env` içinde `EXPO_PUBLIC_SUPABASE_*`; script’ler için ayrıca `SUPABASE_SERVICE_ROLE_KEY`.
- EAS: `package.json` içindeki `ios:testflight:build` / `android:preview:build` vb. `scripts/eas-with-vcs.mjs` ile çalışır; ortamda `EAS_NO_VCS=1` olsa bile kaldırılır ve EAS git/commit bilgisini kullanabilir. Ham `npx eas-cli build` yerine bu npm script’leri kullan.

## Portföy ve auth
- Seçili portföy: `context/portfolio.tsx` — `refresh()` boş listede **Ana Portföy** oluşturur; toplu yüklemede `price_updated_at` / oturum uyumu önemli.
- Yeni kullanıcıda varsayılan portföy: `database/migrations/011_new_user_default_portfolio.sql` (Supabase’e uygulanmalı).
- Portföy adı eşlemesi: `lib/portfolio-name-loose.ts`, TEFAS hata kodu **-100**: `lib/fon-price-guards.ts` + `scripts/sync-tefas-funds.js`.

## Günlük % ve saat dilimi
- UI’da gösterilen günlük değişim: `lib/effective-change-24h.ts` + `lib/trading-day-display.ts` — **BIST / fon**: seans sonrası ve hafta sonu boyunca son işlem günü %’i; sıfırlama yalnızca **ertesi işlem günü 00:00 (TSİ)**. **Yurtdışı**: aynı mantık, **America/New_York**. **Kripto / döviz / emtia / mevduat**: **TSİ** gece yarısı; yeni gün fiyatı gelene kadar gizle.
- **Emtia** sunucu tarafı gece referansı: `scripts/emtia-midnight-tr.js` + `sync-emtia-scrape.js` / `sync-kapalicarsi-gold.js`.
- Gece yarısı geçişinde ekranın yenilenmesi: `hooks/use-minute-tick.ts`.

## Sync ve script’ler
- Node script’leri `scripts/` altında; fonksiyonel isimler (`sync-tefas-funds.js`, `reset-for-csv-import.js`, vb.).
- TEFAS (GitHub Actions IP engeli): üretimde `supabase/functions/sync-tefas` + `docs/SUPABASE-TEFAS-EDGE.md` (pg_cron + pg_net). Sync fon tipleri: **YAT, EMK, BYF, GYF, GSYF** (proje GYF’ler `GYF` altında, örn. TN1). Yerel Windows zamanlama (ör. 07:30–12:30): `scripts/windows/register-tefas-morning-task.ps1` + `docs/LOCAL-TEFAS-WINDOWS-SCHEDULE.md`.
- Yerel Windows fiyat scheduler: Portfolio 30 dk + ABD 10 dk için `scripts/windows/register-local-price-sync-tasks.ps1` + `docs/LOCAL-WINDOWS-PRICE-SCHEDULE.md`.
- ABD (yurtdışı) fiyatları: Yahoo GitHub runner’da (`us-sync.yml`); periyot **Supabase** `pg_cron` → Edge `sync-abd-prices` (`github_dispatch`). GitHub’da `schedule` yok: `docs/SUPABASE-ABD-SYNC.md`.
- **Portfolio sync** (kripto/BIST/döviz/emtıa/holdings yurtdışı; sync sırasında PH yazılmaz): `portfolio-sync.yml`; periyot **Supabase** `pg_cron` → Edge `dispatch-portfolio-sync`: `docs/SUPABASE-PORTFOLIO-SYNC.md`.
- **price_history** (2 gün, günde 1 satır/varlık): migration `022_price_history_daily_two_day_retention.sql`; pg_cron **23:55 TSİ** → `run_daily_price_history_maintenance()`; kurulum `npm run setup:daily-price-history`; manuel `npm run daily-price-history`.
- Açılış fiyat yedeği: `lib/asset-last-price-cache.ts` + `lib/portfolio-summary-cache.ts` (telefon); `lib/prune-local-portfolio-cache.ts` aktif portföy/holding dışındaki yerel önbelleği siler.
- Kullanıcı verisini silme/temizlik: `reset-for-csv-import.js` (master `assets` silmez); tam kullanıcı silme: `delete-user-by-email.js`.
- Günlük admin raporu (20:00 TSİ → hasimozturk@gmail.com): Supabase Edge `daily-admin-report` + `node scripts/setup-daily-report-supabase.mjs re_...` — `docs/SUPABASE-DAILY-ADMIN-REPORT.md`.

## Veritabanı
- Şemalar: `database/migrations/`, özet: `scripts/init-db.sql`.
- RLS değişikliklerinde istemci + service role davranışını ayrı düşün.

## UI: alt butonlar (dokunma)
- Birincil CTA’lar (`Devam`, giriş, sosyal giriş, form onayı vb.) **`ScrollView` / `FlatList` içine konmaz**; `components/screen-with-footer.tsx` içindeki **`ScreenWithFooter`** bileşeninin `footer` slot’una verilir.
- `ScreenWithFooter` düzeni: **flex sütun** (`header` → `body` `flex:1` + `minHeight:0` → `footer` `flexShrink:0`). Footer için `absolute` / `zIndex` kullanma; `FlatList`/`ScrollView` gövdede `style={{ flex: 1 }}` olmalı.
- `keyboardAvoid` yalnızca **gövdeyi** sarar; footer dışarıda kalır (klavye + ScrollView alt CTA dokunuş çakışması).
- Sosyal giriş gibi kritik alt CTA’larda `react-native-gesture-handler` `Pressable` tercih et; kökte `GestureHandlerRootView` gerekir.
- Alt safe area yalnızca footer’da `ScreenWithFooter` tarafından uygulanır.

## Bitirmeden önce
- Mümkünse `npx tsc --noEmit`; davranış değiştiyse ilgili ekran veya script akışını gözden geçir.
- Kapsamı gereksiz büyütme; aynı mantığı tekrar etme — ortak `lib/` veya mevcut hook’ları genişlet.

---
> Source: [gulbeyazozturk/portfoy-takip](https://github.com/gulbeyazozturk/portfoy-takip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
