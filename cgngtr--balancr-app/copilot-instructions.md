## balancr-app

> **Balancr**, arkadaşlar, aile ve ev arkadaşları arasında giderleri kolayca bölüşmek için tasarlanmış modern bir web uygulamasıdır. Uygulama Next.js, Tailwind CSS, shadcn/ui komponentleri ve Supabase veritabanı kullanılarak geliştirilmiştir.

---
description: 
globs: *.tsx, *.ts
alwaysApply: true
---
---
description:
globs:
alwaysApply: false
---
# Balancr - Gider Paylaşım Uygulaması Planı

## Proje Analizi

**Balancr**, arkadaşlar, aile ve ev arkadaşları arasında giderleri kolayca bölüşmek için tasarlanmış modern bir web uygulamasıdır. Uygulama Next.js, Tailwind CSS, shadcn/ui komponentleri ve Supabase veritabanı kullanılarak geliştirilmiştir.

NOT: Balancr; web uygulamasından mobil uygulamaya geçiş aşamasındadır. 

### Teknoloji Yığını:
- **Frontend Framework**: Next.js 14 (App Router)
- **UI Komponentleri**: shadcn/ui & next-themes (koyu mod)
- **Styling**: Tailwind CSS
- **Veritabanı & Kimlik Doğrulama**: Supabase
- **Form Yönetimi**: React Hook Form & Zod
- **Animasyonlar**: Framer Motion
- **Grafikler & Görselleştirme**: Recharts

### Ana Özellikler:
1. **Kullanıcı Kimlik Doğrulama:** Kayıt olma, giriş yapma ve e-posta doğrulama
2. **Grup Yönetimi:** Farklı amaçlar için gruplar oluşturma ve yönetme
3. **Gider Ekleme ve Takibi:** Çeşitli kategorilerde gider girişi
4. **Akıllı Gider Bölüşümü:** Eşit, yüzdelik veya sabit miktarlar şeklinde gider paylaşımı
5. **Borç ve Alacak Takibi:** Kişiler arasındaki finansal bakiyelerin görselleştirilmesi
6. **Ödeme Özeti:** Herkesin kime ne kadar borçlu olduğunu gösteren basitleştirilmiş özetler
7. **Koyu/Açık Mod Desteği:** next-themes ile tema değiştirme

### Veritabanı Modeli:
- **users**: Kullanıcı profilleri (Supabase Auth ile bağlantılı)
- **groups**: Gider grupları (örn. ev giderleri, tatil, etkinlikler)
- **group_members**: Grup üyelikleri ve bakiyeler
- **expenses**: Gider kayıtları, ödeme ve bölüşüm detayları

## TODO

- Expo Development Build oluştur:
  - iOS için: EAS Build kullan (`npx eas build -p ios --profile development`) (Windows'ta yerel build mümkün değil).
  - Android için: `npx expo run:android` (Windows'ta mümkün).
- `RevenueCatOfferings` bileşenindeki `handlePurchase` fonksiyonunda Yetki ID'sini (`'premium'`) kendi ID'nizle değiştir.
- Kullanıcının abonelik durumunu kontrol et ve premium özellikleri buna göre yönet (`Purchases.getCustomerInfo`).

### Premium Özellikleri Roadmap

#### Aşama 0: UI Tasarım ve Prototipler (2-3 hafta)
- [ ] Premium özelliklerin UI wireframe ve mockup'larını oluştur
- [ ] İstatistikler sayfası için detaylı UI tasarımı
- [ ] Grup premium paylaşım ekranları için UI tasarımı
- [ ] Özelleştirme seçenekleri (temalar, renkler, dashboard layout) için UI tasarımı
- [ ] Entegrasyon ekranları (banka hesapları, makbuz tarama) için UI tasarımı
- [ ] Premium üyelik satın alma akışını iyileştirme tasarımı
- [ ] Premium özelliklerin devre dışı/kısıtlı versiyonlarının ücretsiz kullanıcılarda nasıl görüneceğinin tasarımı
- [ ] Figma veya benzeri araçlarda etkileşimli prototip oluşturma
- [ ] Tasarımlar için kullanıcı geri bildirimi toplama ve iyileştirme

#### Aşama 1: Altyapı Hazırlığı (1-2 hafta)
- [x] Premium özellik kontrol mekanizması oluştur (`hooks/usePremium.ts`)
- [x] Premium UI göstergeleri için temalar ve bileşenler oluştur
- [x] RevenueCat entegrasyonunu tamamla ve ödeme akışını test et
- [x] Premium durum değişikliklerini yönetecek context/store yapısını oluştur

#### Aşama 2: Gelişmiş İstatistikler (2-3 hafta)
- [ ] İstatistikler sayfası tasarımı ve geliştirilmesi
- [ ] Kategori ve zaman bazlı harcama analizleri geliştir
- [ ] Grafikler ve görsel raporlar için chart bileşenleri oluştur
- [ ] PDF/Excel formatında rapor ihracı özelliği ekle
- [ ] Premium kullanıcılar için AI destekli gelecek harcama tahminleri ekle

#### Aşama 3: Grup Paylaşım Modeli (2 hafta)
- [ ] Grup içinde premium üyelik maliyetini paylaşma sistemini geliştir
- [ ] Grup premium özellikleri yönetmek için backend yapısı oluştur
- [ ] Grup premium durumunu yönetmek için UI geliştir
- [ ] Grup içi premium teklif sistemi için bildirim mekanizması ekle

#### Aşama 4: Özelleştirme ve Arayüz İyileştirmeleri (1-2 hafta)
- [ ] Premium kullanıcılar için özel temalar geliştir
- [ ] Özelleştirilebilir gösterge panelleri oluştur
- [ ] Gelişmiş navigasyon seçenekleri ekle
- [ ] Premium kullanıcılar için özel animasyonlar ekle

#### Aşama 5: Entegrasyonlar (2-3 hafta)
- [ ] Banka hesapları entegrasyonu için API araştırması ve güvenlik yapısı
- [ ] Daha fazla ödeme platformu entegrasyonu
- [ ] Makbuz tarama ve OCR entegrasyonu
- [ ] Otomatik gider girişi için AI tabanlı kategorileştirme

#### Aşama 6: Test ve Optimizasyon (1-2 hafta)
- [ ] A/B testleri için altyapı oluştur
- [ ] Kullanıcı yolculuğu analizi ve iyileştirmeler
- [ ] Premium özellik kullanım analitiği ekle
- [ ] Performans optimizasyonları

#### Aşama 7: Pazarlama ve Büyüme (Sürekli)
- [ ] Ücretsiz deneme süresi için altyapı oluştur
- [ ] Referans programı ve grup indirimleri için sistem geliştir
- [ ] Başarı hikayelerini ve sosyal kanıtları gösteren UI ekle
- [ ] Kullanıcılara yönelik demo ve öğretici içerikler hazırla

## DONE

- Satın almaları geri yükleme ("Restore Purchases") fonksiyonunu `RevenueCatOfferings` bileşenine ekle (`Purchases.restorePurchases`).
- Development Build'de görüntülenmeyen metinleri düzelt (Text ve SectionHeader bileşenlerindeki Theme kullanımını düzelt).
- Satın alma işlemini tetikleme mantığını (`Purchases.purchasePackage`) ve hata yönetimini `RevenueCatOfferings` bileşenine ekle.
- `premium` ekranını `app/(tabs)/_layout.tsx` dosyasında tab bar'dan gizle.
- `RevenueCatOfferings` bileşenini `app/(tabs)/premium.tsx` ekranına ekle.
- `app/(tabs)/profile.tsx` ekranına "Premium'a Yükselt" butonu ekle.
- `app/(tabs)/profile.tsx` dosyasındaki Text variant lint hatasını düzelt.
- `context/ThemeContext.tsx` dosyasındaki lint hatasını düzelt.
- RevenueCat Ürünlerini (Products) ve Tekliflerini (Offerings) almak için `RevenueCatOfferings.tsx` bileşenini oluştur.
- Metro Bundler'ı yeniden başlatarak `.env` değişikliklerinin yüklenmesini sağla.
- RevenueCat API anahtarını `.env` dosyasına ekle ve `App.tsx`'i güncelle.
- `react-native-purchases` paketini npm veya yarn ile yükle.
- Gerekirse iOS için `pod install` çalıştır. (Expo Managed olduğu için atlandı)
- RevenueCat SDK'sını başlat (Initialize). (App.tsx)
- Premium özellik kontrol mekanizması oluştur (`PremiumContext.tsx`)
- Premium UI göstergeleri için temalar ve bileşenler oluştur (`theme/premium.ts`, `PremiumBadge`, `PremiumCard`, `PremiumLock`)
- RevenueCat entegrasyonunu tamamla ve ödeme akışını test et
- Premium durum değişikliklerini yönetecek context/store yapısını oluştur
- Tüm premium özellikleri içeren ekran yönlendirmeleri eklendi

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cgngtr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
