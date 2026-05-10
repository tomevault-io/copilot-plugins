## multilanguage-rules

> Proje zaten tam fonksiyonel i18next implementasyonuna sahip:

# React Native CLI Boilerplate - i18n Çeviri Kuralları

## Mevcut i18n Yapısı (i18next)
Proje zaten tam fonksiyonel i18next implementasyonuna sahip:

```
src/
├── config/
│   └── i18n.ts              # i18next yapılandırması
├── hooks/
│   └── useI18n.ts           # Çeviri hook'u (formatters dahil)
├── components/
│   └── I18nProvider.tsx     # i18n Provider wrapper
└── locales/
    ├── tr.json              # Türkçe çeviriler (varsayılan)
    └── en.json              # İngilizce çeviriler (fallback)
```

## 1. Translation File Structure (Mevcut Yapı)
- Çeviriler `src/locales/` klasöründe JSON formatında saklanır
- **tr.json:** Türkçe çeviriler (varsayılan dil)
- **en.json:** İngilizce çeviriler (fallback dil)
- **Nested structure kullanılır:** `common.loading`, `auth.loginSuccess`

## 2. useI18n Hook Usage (Mevcut)
```typescript
import { useI18n } from '@hooks/useI18n';

const MyComponent = () => {
  const { 
    t,                    // Çeviri fonksiyonu
    changeLanguage,       // Dil değiştirme
    getCurrentLanguage,   // Mevcut dil
    formatDate,          // Tarih formatlama (tr-TR/en-US)
    formatNumber,        // Sayı formatlama  
    formatCurrency,      // Para formatlama
    currentLanguage,     // Mevcut dil (computed)
    availableLanguages   // Mevcut diller (computed)
  } = useI18n();

  return <Text>{t('common.loading')}</Text>;
};
```

## 3. Mevcut Key Structure'ı Koruyun
Proje zaten iyi organize edilmiş nested structure kullanıyor:

```json
{
  "common": { ... },          // Genel UI elementleri
  "navigation": { ... },      // Navigasyon metinleri  
  "auth": { ... },           // Kimlik doğrulama
  "profile": { ... },        // Profil yönetimi
  "settings": { ... },       // Ayarlar
  "notifications": { ... },  // Bildirimler
  "errors": { ... },         // Hata mesajları
  "validation": { ... },     // Form validasyonları
  "onboarding": { ... },     // Onboarding süreçleri
  "gender": { ... },         // Cinsiyet seçimi
  "age": { ... },           // Yaş seçimi
  "onesignal": { ... }      // OneSignal entegrasyonu
}
```

## 4. Translation Key Usage (Mevcut Pattern)
```typescript
// Nested object structure kullanın
t('common.loading')              // "Yükleniyor..."
t('auth.loginSuccess')           // "Giriş başarılı"
t('gender.gender_screen_title')  // "Cinsiyetinizi seçin"
t('age.age_screen_title_part1')  // "Kaç "
```

## 5. Interpolation (i18next Style)
```typescript
// JSON'da placeholder format:
{
  "welcome_message": "Hoş geldin {{name}}!",
  "selected_age": "Seçilen yaş: {{age}}"
}

// Component'te kullanım:
t('welcome_message', { name: userName })
t('age.selected_age', { age: selectedAge })
```

## 6. Language Switching Implementation
```typescript
const LanguageSwitcher = () => {
  const { changeLanguage, currentLanguage } = useI18n();
  
  return (
    <View>
      <TouchableOpacity onPress={() => changeLanguage('tr')}>
        <Text>Türkçe</Text>
      </TouchableOpacity>
      <TouchableOpacity onPress={() => changeLanguage('en')}>
        <Text>English</Text>
      </TouchableOpacity>
    </View>
  );
};
```

## 7. Advanced Features (Mevcut Hook'ta)
```typescript
const { formatDate, formatNumber, formatCurrency } = useI18n();

// Locale-aware formatting
formatDate(new Date(), { year: 'numeric', month: 'long' });
formatNumber(1234.56); 
formatCurrency(1000, 'TRY');
```

## 8. File Synchronization Rules
Her yeni key MUTLAKA her iki dil dosyasında olmalı:
1. tr.json'a ekle
2. en.json'a ekle  
3. Her iki dosyada da aynı nested path'te olduğunu doğrula

## 9. Development Workflow
Yeni çeviri eklerken adımlar:
1. **Key İsimlendirme:** Mevcut pattern'e uygun isim belirle
2. **Section Belirleme:** Hangi section'a ait olduğunu belirle
3. **tr.json Güncelleme:** Türkçe çeviriyi ekle
4. **en.json Güncelleme:** İngilizce çeviriyi ekle
5. **Component Update:** useI18n hook'unu component'te kullan
6. **Test:** Her iki dilde de test et

## 10. Code Review Checklist
- [ ] Yeni key'ler her iki dil dosyasında mevcut
- [ ] Nested structure doğru şekilde organize
- [ ] Interpolation placeholder'ları tutarlı
- [ ] Component'lerde hardcode string yok
- [ ] UI layout her iki dilde de çalışıyor

## 11. Example Implementation
```typescript
// Correct usage with current structure
import { useI18n } from '@hooks/useI18n';

const MyComponent = () => {
  const { t } = useI18n();
  
  return (
    <View>
      <Text>{t('common.loading')}</Text>
      <Text>{t('errors.networkError')}</Text>
      <Text>{t('auth.welcome')}</Text>
    </View>
  );
};
``` 

---
> Source: [hasaneyldrm/reactnativecliboilerplate](https://github.com/hasaneyldrm/reactnativecliboilerplate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
