## x-unfollow-radar

> Bu projede (X Unfollow Radar) kullanılan kod stili ve sözleşmeler.


# Kod stili – X Unfollow Radar

Bu kural **yalnızca bu proje** içinde geçerlidir. Yeni veya değiştirilen kod bu stile uymalıdır.

## Genel

- **Dil**: Vanilla JavaScript (ES6+), framework yok.
- **Strict**: Her modülün başında `'use strict';` kullan.
- **Girinti**: 4 boşluk.
- **Tırnak**: Tek tırnak (`'`) tercih et (string'ler için).

## Modül yapısı

- Modüller **IIFE** ile sarılı, tek bir global namespace dönsün.
- Namespace adı PascalCase (örn. `XUnfollowRadarContent`, `Constants`).
- Dosya başında `@fileoverview` ve `@description` içeren JSDoc bloğu olsun.

```javascript
/**
 * @fileoverview Modülün kısa açıklaması
 * @description Detaylı açıklama
 */

const MyModule = (function () {
    'use strict';

    // ...
    return { publicMethod };
})();
```

## Bölümleme ve yorumlar

- Uzun dosyalarda bölümler için çift çizgiyle ayrılmış blok kullan:

```javascript
// ═══════════════════════════════════════════════════════════════
// BÖLÜM ADI (örn. PRIVATE STATE, PRIVATE METHODS)
// ═══════════════════════════════════════════════════════════════
```

- Fonksiyonlar ve önemli sabitler için JSDoc kullan: `@param`, `@returns`, `@type` (gerekirse).

## İsimlendirme

- **Sabitler / config**: `UPPER_SNAKE_CASE`; config objeleri `Object.freeze()` ile dondur.
- **Değişkenler ve fonksiyonlar**: `camelCase`.
- **Namespace / “sınıf” benzeri**: `PascalCase`.
- Kod ve yorumlarda **İngilizce** terim kullan; UI metinleri i18n (`data-i18n`, `locales`) ile.

## Sabitler ve config

- Sihirli sayılar ve seçiciler `src/shared/constants.js` içinde toplanmalı; doğrudan `Constants.*` referansı kullan.

```javascript
// İyi
await randomDelay(Constants.TIMING.MIN_DELAY, Constants.TIMING.MAX_DELAY);

// Kaçınılacak
await randomDelay(2000, 5000);
```

## Asenkron kod

- `async/await` tercih et; gerekmedikçe callback zinciri kullanma.
- Promise'ler için tutarlı hata yakalama (try/catch veya `.catch()`).

## Event ve DOM

- Event listener’lar için `addEventListener` kullan; inline `onclick` vb. kullanma.
- DOM seçicileri mümkün olduğunca `Constants` (SELECTORS, vb.) üzerinden gelsin.

---

Bu stile uyarak yazılan kod, mevcut codebase ile tutarlı kalır ve sadece bu projede geçerli olur.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/berkayismus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
