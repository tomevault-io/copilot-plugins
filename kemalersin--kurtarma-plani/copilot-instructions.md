## banking-preset

> TCMB/bankacılık referans preset — embed, IndexedDB, çevrimiçi güncelleme


# Bankacılık Referans Preset

## Katmanlar

1. **Embed** — build ile `src/data/banking-presets/tr-*.json` gömülür
2. **IndexedDB** — `bankingPreset` store; aktif kayıt
3. **Okuma önceliği** — DB varsa DB, yoksa embed

## Çevrimiçi güncelleme

- Yalnızca kullanıcı "Güncelle" dediğinde ve `navigator.onLine`
- `fetch(feedUrl)` → JSON → `BankingPresetSchema` (Zod)
- Başarı: `put` ile DB kaydını **ez**; `source: 'remote'`, `fetchedAt` ISO
- Hata: embed/DB korunur; kullanıcıya toast

## Manuel import

- Dosya seçici → aynı Zod şema → DB ezme (`source: 'import'`)

## Form kullanımı

- Düğme metni daima **"Referansla doldur"** (kullanıcıya `preset` terimini gösterme); örn. `LoanFormDrawer` KKDF+BSMV alanı
- Yalnızca preset okur; kullanıcı alanları sonra düzenler
- Hesap motoru kullanıcı/sözleşme oranını kullanır, preset'i değil

## Şema

- `id`, `label`, `effectiveFrom`, `schemaVersion`, ürün blokları (`creditCard`, …)
- Uzak feed ile embed **aynı şema**

---
> Source: [kemalersin/kurtarma-plani](https://github.com/kemalersin/kurtarma-plani) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
