## app-rules

> - React ile TypeScript kullanarak mobil öncelikli bir web uygulaması geliştirin

# Mobil Öncelikli React Uygulaması Kuralları

## Proje Genel Kuralları

### Dil ve Çerçeve
- React ile TypeScript kullanarak mobil öncelikli bir web uygulaması geliştirin
- Backend için Node.js ve Express kullanın
- Veritabanı: SQLite kullanın, veritabanı dosyası proje kök dizininde database.db olarak saklansın

### Mobil Öncelikli Tasarım
- Tüm UI tasarımları mobil öncelikli olmalı
- Tailwind CSS ile responsive tasarım uygulanmalı
- min-width yerine max-width medya sorguları kullanın

### Telefon Numarası Benzersizliği
- Müşteri telefon numaraları benzersiz olmalı
- Veritabanında UNIQUE kısıtlaması ile kontrol edilsin

### Dosya Yapısı
- Özellik bazlı dosya yapısı kullanın
- Müşteri yönetimi için src/features/customers
- Sipariş yönetimi için src/features/orders

## Frontend Kuralları

### React
- Fonksiyonel bileşenler ve hook'lar kullanın
- Bileşen isimleri PascalCase olmalı (ör. CustomerList)
- Prop'lar için TypeScript tipleri tanımlayın
- Durum yönetimi için useState ve useReducer kullanın
- Global durum için useContext veya Zustand tercih edin

### UI Kütüphaneleri
- Tailwind CSS ile stil oluşturun, utility-first yaklaşımı benimseyin
- Erişilebilirlik (a11y) için semantik HTML kullanın
- Mobil cihazlarda düzgün çalışması için max-width medya sorguları kullanın

### Performans
- Gereksiz yeniden render'ları önlemek için React.memo kullanın
- Büyük veri listeleri için React.lazy ve Suspense ile kod bölme uygulayın

## Backend Kuralları

### Node.js ve Express
- RESTful API endpoint'leri oluşturun (ör. /api/customers, /api/orders)
- Hata yönetimi için özel hata sınıfları kullanın
- Giriş doğrulaması için Zod kullanın

### SQLite
- Müşteri tablosu (customers): id (PRIMARY KEY), name (TEXT), phone (TEXT, UNIQUE)
- Sipariş tablosu (orders): id (PRIMARY KEY), customerId (FOREIGN KEY), deliveryTime (TEXT, 'morning' veya 'evening'), orderDate (TEXT, ISO format), status (TEXT, 'pending', 'delivered', vb.)
- Veritabanı işlemleri için knex.js kullanın
- INSERT OR REPLACE ile müşteri güncellemelerini yönetin

## Özellik Gereksinimleri

### Müşteri Yönetimi
- Ekleme: Yeni müşteri eklerken telefon numarasının benzersizliğini kontrol edin
- Silme: Müşteri silindiğinde ilgili siparişler de silinsin
- Güncelleme: Müşteri adı veya telefon numarasını güncelleyin

### Sipariş Yönetimi
- Sipariş eklerken müşteri ID'si ve teslimat saati (morning veya evening) zorunlu olmalı
- Sipariş durumu dashboard'da görselleştirilsin (ör. bekleyen, teslim edilmiş)

### Dashboard
- Günlük siparişleri tarih ve teslimat saatine göre listeleyin
- Müşteri bazlı sipariş özeti gösterin
- Basit istatistikler (ör. toplam sipariş sayısı) sunun

## Erişilebilirlik ve Güvenlik
- SQL enjeksiyonunu önlemek için parametreli sorgular kullanın
- Erişilebilirlik için ARIA etiketlerini gerektiğinde ekleyin

---
> Source: [astopaal/meksut-order-management](https://github.com/astopaal/meksut-order-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
