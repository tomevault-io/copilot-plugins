## konak-kebap-open-tab

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# Next.js 16 — dikkat

Bu proje Next.js 16.2.6 + React 19 kullanır; eğitim verindeki eski sürümlerden
farklı olabilir. Next'e özgü bir API'ye (routing, route handler, server/client
component, `next/*` importları, config) dokunmadan ÖNCE
`node_modules/next/dist/docs/` içindeki ilgili rehberi oku. Sadece bizim kodumuzu
düzenlerken bu gerekmez.
<!-- END:nextjs-agent-rules -->

# Proje: Konak Kebap — Tedarik & Cari Takip

Restoranın toptancı/tedarikçi alışlarını, ödemelerini ve cari borcunu takip eder.

## Stack
Next.js 16 (App Router, Turbopack) · React 19 · TypeScript · Prisma 7 (driver
adapter `@prisma/adapter-pg`) · Neon Postgres · Zod 4 · Tailwind 4.

## Komutlar
- `npm run dev` — geliştirme sunucusu
- `npm run db:migrate` — şema değişince migration oluştur+uygula (`prisma migrate dev`)
- `npm run db:seed` — örnek veri
- `npm run db:studio` — DB'yi görsel incele
- `npm run lint` / `npm run build`

## Değişmez kurallar (bunlara uy)
- **Para HER ZAMAN kuruş (integer).** DB'de, hesapta, transferde kuruş tut.
  TL ↔ kuruş dönüşümü ve biçimlendirme yalnızca [src/lib/money.ts](src/lib/money.ts) üzerinden.
- **Soft delete:** kayıtlar silinmez, `deletedAt` set edilir. Sorgularda
  `deletedAt: null` filtresini unutma.
- **PurchaseItem.unitPrice DONDURULUR** — alış kalemine yazılan fiyat sonradan değişmez.
- **API zarfı:** her route `{ data }` veya `{ error }` döner. Helper'ları kullan:
  `ok / created / fail / handleError` ([src/lib/api.ts](src/lib/api.ts)).
- **Girdi doğrulama:** Zod şemaları [src/lib/validations.ts](src/lib/validations.ts) içinde.
- Import alias `@/*` → `src/*`.

## Harita
- `prisma/schema.prisma` — veri modeli (Supplier, Product, ProductPackage,
  Purchase, PurchaseItem, Payment, PriceHistory). Generated client `src/generated/prisma` (gitignore).
- `src/lib/` — `prisma` (client), `money`, `balance` (cari hesap), `api`, `validations`.
- `src/app/api/` — route handler'lar. `src/app/` — sayfalar.

## Notlar
- Türkçe yaz (kod yorumları, UI, kullanıcı mesajları).
- Gizli değerler `.env`'de (gitignore). Asla commit etme/loglaama.
- Şu an direct Neon bağlantısı kullanılıyor; Vercel deploy'da pooled string'e geç.

---
> Source: [Metinkeles1/konak-kebap-open-tab](https://github.com/Metinkeles1/konak-kebap-open-tab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
