## event-pulse-platform

> Bu dosyayı **sana verdiğim talimat** gibi düşün: Bu repoda çalışırken aşağıdakilere uy; gerektiğinde kök `README.md` ve `docs/` ile çapraz kontrol et.

# EventPulse — Cursor / AI için kurallarım

Bu dosyayı **sana verdiğim talimat** gibi düşün: Bu repoda çalışırken aşağıdakilere uy; gerektiğinde kök `README.md` ve `docs/` ile çapraz kontrol et.

## Teknoloji yığını

- **Backend:** Node.js, TypeScript, Fastify.
- **Veri:** PostgreSQL (TimescaleDB), Redis.
- **Kuyruk:** Redis Streams.
- **Test:** Vitest (bu projede Jest değil).

## Kodlama

- Mümkün olduğunca **saf fonksiyon** ve okunabilir akış; gereksiz soyutlama ekleme.
- **TypeScript:** `any` kullanma; veri yapıları için açık `interface` / `type`.
- **Async:** `async/await`; hata yollarını unutma (try/catch veya Fastify’un hata modeli).
- **Doğrulama:** Runtime şema için Zod (veya projede kullanılan eşdeğer) — API gövdeleri ve sınırlar net olsun.
- **Log:** Yapılandırılmış log (pino); hassas başlıkları loglama (projede zaten redaksiyon var).

## Mimari

- **Katmanlar:** HTTP/route → use case / servis → repository / worker; Clean Architecture dizinlerine uy.
- **Olay güdümlü:** Ingestion’da mümkün olduğunca hızlı **202 Accepted**; ağır iş worker + kuyrukta.
- **Dayanıklılık:** Stream tüketicide yeniden deneme (ör. **en fazla 3** deneme sonrası DLQ) — mevcut kodla çelişme.

## Dokümantasyon (önemli)

- Anlamlı bir özellik veya mimari değişiklik yaptığında, özeti **`docs/ai-log.md`** içinde mevcut şablona uygun şekilde (**AI-NNN** başlığı, Category, Context, Prompt, Output, **Your Modifications**, Validation) işlemeyi unutma.
- Ben bir şeyi düzeltip seni yönlendirdiysem, bunu günlükte **Your Modifications** altında net yaz.

## Dil

- Benimle konuşurken yanıtları **Türkçe** tercih ediyorum; kod ve commit mesajında proje diline uy.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MuhammetKeskinDev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
