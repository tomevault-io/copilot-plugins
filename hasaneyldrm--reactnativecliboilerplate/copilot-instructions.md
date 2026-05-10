## general-rules

> Her dosya amacına göre belirlenen klasörde bulunmalıdır. Bu, projenin öngörülebilir ve düzenli kalmasını sağlar.


# React Native CLI Boilerplate - Proje Kuralları

## Rule 1: The Sacred Folder Structure (Separation of Concerns)
Her dosya amacına göre belirlenen klasörde bulunmalıdır. Bu, projenin öngörülebilir ve düzenli kalmasını sağlar.

**Mevcut Proje Yapısı:**
```
src/
├── assets/          # Resimler, fontlar, animasyonlar (logo.png, lottie/)
├── components/      # Yeniden kullanılabilir UI bileşenleri (I18nProvider, OneSignalProvider, vb.)
├── config/          # Yapılandırma dosyaları (i18n.ts, onesignal.ts, storage.ts)
├── hooks/           # Özel React hook'ları (useI18n.ts, useOneSignal.ts, useStorage.ts)
├── locales/         # Çeviri dosyaları (tr.json, en.json)
├── navigation/      # React Navigation mantığı (RootNavigator.tsx)
├── onboarding/      # Onboarding ekranları (AgeScreen.tsx, GenderScreen.tsx)
├── screens/         # Ana ekran bileşenleri (FirstScreen/)
└── utils/           # Yardımcı fonksiyonlar (storage.ts)
```

**Gelecekte Eklenecek Klasörler:**
```
src/
├── api/             # API istemcileri ve çağrıları (ihtiyaç halinde)
├── constants/       # Uygulama genelindeki sabitler (ihtiyaç halinde)
├── store/           # Redux Toolkit durum yönetimi (ihtiyaç halinde)
├── theme/           # Stil ve tema (renkler, fontlar, boşluklar)
└── types/           # Global TypeScript tipleri
```

## Rule 2: Technology Stack (Mevcut)
**Zaten Kurulu Teknolojiler:**
- **Navigation:** React Navigation v7 (native-stack, stack)
- **Styling:** styled-components v6
- **Internationalization:** i18next + react-i18next
- **Animations:** lottie-react-native
- **Icons/Graphics:** react-native-svg
- **TypeScript:** Full type safety

## Rule 3: Absolute Imports (Zaten Yapılandırılmış)
tsconfig.json ve babel.config.js zaten doğru şekilde yapılandırılmış:
- `@/*` → `src/*`
- `@assets/*` → `src/assets/*`
- `@components/*` → `src/components/*`
- `@config/*` → `src/config/*`
- `@hooks/*` → `src/hooks/*`
- `@navigation/*` → `src/navigation/*`
- `@screens/*` → `src/screens/*`
- `@utils/*` → `src/utils/*`

## Rule 4: Modular Navigation (Zaten Uygulanmış)
App.tsx dosyası temiz tutulmuş:
```typescript
<I18nProvider>
  <NavigationContainer>
    <RootNavigator />
  </NavigationContainer>
</I18nProvider>
```

## Rule 5: Consistent Styling
styled-components zaten projeye entegre edilmiş ve kullanılıyor.
StyleSheet.create yerine styled-components tercih edilecek.

## Rule 6: State Management Strategy
**Mevcut:** Local state + React hooks
**Gelecek:** Gerektiğinde Redux Toolkit eklenecek (Context API global state için yasak)

## Rule 7: Clean Code Discipline
- **Strict Typing:** `any` tipinden kaçın
- **Descriptive Naming:** Anlamlı isimler kullan
- **Component Architecture:** Functional components kullan
- **Props Interface:** Her component için interface tanımla

---
> Source: [hasaneyldrm/reactnativecliboilerplate](https://github.com/hasaneyldrm/reactnativecliboilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
