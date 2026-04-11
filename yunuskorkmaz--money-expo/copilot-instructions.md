## money-expo

> Bu dosya Gemini için proje talimatlarını içerir.

# Money Expo - Gemini Instructions

Bu dosya Gemini için proje talimatlarını içerir.

## Proje Özeti

Money Expo, offline-first bir kişisel finans takip uygulamasıdır.
- **Stack**: Expo + TypeScript + Drizzle ORM + SQLite
- **Migration**: Drizzle migration-first (`drizzle/migrations`)
- **Dil**: Türkçe arayüz
- **Theme**: Dark

## Bağlam Dökümanları

Bu dosyayı okurken aşağıdaki dökümanları da referans al:

1. **AGENTS.md** - Kod stili, build komutları, proje yapısı
2. **PRD.md** - Ürün gereksinimleri ve iş kuralları (BR-01..BR-08)
3. **VIBE_CODING_SPEC.md** - Detaylı UI/UX spesifikasyonu
4. **.opencode/ORCHESTRATION.md** - Agent koordinasyonu

## Agent Sistemi

Bu proje uzmanlaşmış agent'lar ile çalışır. Task türüne göre ilgili agent'ı kullan:

| Task | Agent Dosyası |
|------|---------------|
| Schema/Migration | `.opencode/agents/database-agent.md` |
| Routing/Layout | `.opencode/agents/navigation-agent.md` |
| Form/Validasyon | `.opencode/agents/form-agent.md` |
| Ödeme mantığı | `.opencode/agents/payment-agent.md` |
| Taksit mantığı | `.opencode/agents/installment-agent.md` |
| Ekran UI | `.opencode/agents/ui-agent.md` |

## Kod Standartları

### Import Sırası
```typescript
// 1. React/React Native
import { useState } from 'react';
import { View, Text } from 'react-native';

// 2. Third-party
import { useRouter } from 'expo-router';
import { eq } from 'drizzle-orm';

// 3. Local
import { schema } from '@/src/db/schema';
```

### Type Inference
```typescript
// Schema'dan type türet
export type Payment = typeof payments.$inferSelect;
export type NewPayment = typeof payments.$inferInsert;
```

### Renkler (Premium Dark — Revolut-Style)
```typescript
const colors = {
  background: '#09090b',       // Near-black (Zinc-950)
  surface: '#141418',          // Dark surface
  'surface-lighter': '#1e1e24', // Elevated surface
  'surface-dark': '#0e0e12',   // Deeper surface
  card: '#141418',
  sheet: '#0c0c10',            // Bottom sheet bg
  'input-border': '#27272a',   // Zinc-800
  muted: '#71717a',            // Zinc-500 (muted text)
  primary: '#06f9a0',          // Brand green accent
  success: '#06f9a0',
  warning: '#FFB800',
  danger: '#FF4B4B',
  info: '#4B8BFF',
};
```

### Animasyonlar
- `react-native-reanimated` aktif (babel plugin + doğrudan kullanım)
- Butonlarda `scale` press efekti (0.97 spring)
- Input hatalarında `shake` animasyonu
- Ekranlarda `FadeIn` / `FadeInDown` entering animasyonları
- Kartlarda `boxShadow` ile derinlik, `borderCurve: 'continuous'`

## İş Kuralları

| ID | Kural |
|----|-------|
| BR-01 | `(payment_id, period_key)` UNIQUE |
| BR-02 | Variable amount = installments sum |
| BR-03 | Credit card paid → cascade installments |
| BR-04 | Skipped = excluded from stats |
| BR-05 | Archived = no new transactions |
| BR-06 | Payment delete → cascade transactions + installments + plans |
| BR-07 | Installment only links to `variable` payment |
| BR-08 | `period_key` local-time `"YYYY-MM"` format |

## Build Komutları

```bash
npm start              # Metro bundler
npm run android        # Android build
npx tsc --noEmit       # Type check
npx drizzle-kit generate  # Migration oluştur
```

## Kritik Kurallar

1. **Türkçe**: Tüm kullanıcı metinleri Türkçe
2. **Offline-first**: Ağ gerektiren kod yok
3. **Confirmation**: Silme işlemlerinde onay
4. **Empty state**: Emoji + açıklama
5. **Safe area**: `useSafeAreaInsets()`
6. **Routing**: `router.push(... as any)` kullanma; typed params kullan
7. **UI**: NativeWind (`className`) standardını koru
8. **Animasyonlar**: `react-native-reanimated` ile entering/layout animasyonları
9. **Kartlar**: Border yerine soft shadow + `borderRadius: 16` + `borderCurve: 'continuous'`
10. **Shadows**: CSS `boxShadow` kullan, legacy shadow/elevation kullanma

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yunuskorkmaz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yunuskorkmaz)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
