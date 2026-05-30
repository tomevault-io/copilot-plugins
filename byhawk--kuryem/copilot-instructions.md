## kuryem

> Bu dosya, Claude Code (claude.ai/code) için bu depoda çalışırken rehberlik sağlar.

# CLAUDE.md

Bu dosya, Claude Code (claude.ai/code) için bu depoda çalışırken rehberlik sağlar.

## Proje Genel Bakış

Bu, hem bireysel hem de kurumsal müşteriler için tasarlanmış bir kurye ve lojistik web sistemidir. Sistem yalnızca web tabanlıdır (başlangıçta mobil uygulama yok) ve mikroservislere evrilebilecek modüler monolit mimari izler.

**Hedef Kullanıcılar:**
- Bireysel müşteriler (tek seferlik gönderi)
- Kurumsal müşteriler (abonelik, kontrat, toplu gönderi)

**Temel Platformlar:**
- Müşteri web arayüzü (bireysel & kurumsal)
- Admin/Operasyon paneli
- Kurye responsive mobil web arayüzü

## Domain Mimarisi (Bounded Context'ler)

Sistem, Domain-Driven Design (DDD) prensiplerine göre şu context'lerle yapılandırılmıştır:

1. **Customer Context (Müşteri)** - Bireysel ve kurumsal hesaplar, adres defteri
2. **Order/Shipment Context (Sipariş/Gönderi)** - Sipariş oluşturma, paket bilgisi, takip, durum yaşam döngüsü
3. **Dispatch/Fleet Context (Sevk/Filo)** - Kurye profilleri, araçlar, vardiyalar, atama algoritmaları
4. **Tracking Context (Takip)** - Gönderi olay akışları (alındı, yolda, teslim edildi, başarısız)
5. **Billing Context (Faturalama)** - Gerçek zamanlı fiyat hesaplama, kontrat fiyatları, aylık faturalar
6. **Pricing & Rates Context (Fiyatlama)** - Bölge bazlı tarifeler, ağırlık, paket tipi, hizmet seviyeleri
7. **Notification Context (Bildirimler)** - SMS, e-posta, kurumsal entegrasyonlar için webhook'lar
8. **Integration Context (Entegrasyonlar)** - Harita/rotalama, ödeme gateway'leri, muhasebe/e-fatura
9. **Admin/Reporting Context (Yönetim/Raporlama)** - Dashboard, KPI'lar, raporlama, audit log'ları

Her context, net sınırlarla API üzerinden iletişim kurar. Modüler monolit ile başlanır; gerektiğinde mikroservislere evrilebilir.

## Temel Veri Modeli

### Birincil Varlıklar

**users** - Uygulama kullanıcıları (müşteriler, kuryeler, adminler)
- `type`: individual, corporate_user, admin, courier
- `company_id`: nullable, kurumsal hesaplara bağlantı
- `roles`: RBAC için JSONB

**companies** - Kurumsal hesaplar
- tax_number, fatura bilgisi, credit_limit, billing_cycle

**shipments** - Temel sipariş varlığı
- `tracking_code`: benzersiz tanımlayıcı
- `status`: PENDING → ASSIGNED → IN_TRANSIT → DELIVERED/FAILED/RETURNED
- gönderici/alıcı adres referansları
- `service_type`: normal, express, vip, aracli
- `assigned_courier_id`, `assigned_vehicle_id`

**shipment_events** - Durum değişiklikleri için event sourcing
- Tüm geçişleri aktör, zaman damgası, metadata ile loglar

**Önemli İndeksler:**
- `shipments(tracking_code)` - unique
- `shipments(status, assigned_courier_id)`
- `shipments(created_at)` - zaman bazlı partitioning için
- `shipment_events(shipment_id, created_at)`

## Sipariş İş Akışı (Durum Makinesi)

**Durum Akışı:**
```
PENDING → ASSIGNED → IN_TRANSIT → DELIVERED
                                → FAILED
                                → RETURNED
```

**Geçişler:**
- `create_order` → PENDING
- `admin_assign` / `auto_assign` → ASSIGNED
- `courier_pickup` → IN_TRANSIT
- `courier_deliver` → DELIVERED
- `courier_fail` → FAILED
- `return_processed` → RETURNED

Tüm geçişler `shipment_events` tablosuna loglanmalıdır.

## API Yapısı

Base path: `/api/v1`

**Kimlik Doğrulama:**
- `POST /api/v1/auth/login` - JWT tabanlı (access + refresh token'lar)
- `POST /api/v1/auth/refresh`

**Gönderiler:**
- `POST /api/v1/shipments` - Sipariş oluştur
- `GET /api/v1/shipments/{id}` - Detay getir
- `GET /api/v1/shipments?status=ASSIGNED` - Filtrelenmiş liste
- `POST /api/v1/shipments/{id}/assign` - Kurye ata
- `POST /api/v1/shipments/{id}/events` - Durum güncelle

**Takip:**
- `GET /track/{tracking_code}` - Public takip endpoint'i

**Kuryeler:**
- `GET /api/v1/couriers/assigned` - Kuryenin atanmış görevleri
- `POST /api/v1/couriers/{id}/status` - Konum/durum raporu

**Webhook'lar:**
- `POST /api/v1/webhooks/register` - Kurumsal webhook kaydı

## Kimlik Doğrulama & Yetkilendirme

**RBAC Rolleri:**
- `superadmin` - Tam sistem erişimi
- `operations` - Sevk ve atama
- `dispatcher` - Kurye ataması
- `accounting` - Faturalama ve faturalar
- `courier` - Teslimat görevleri
- `corporate_user` - Kurumsal hesap kullanıcısı
- `individual_user` - Tek gönderi müşterisi

**Güvenlik:**
- JWT token'lar (kısa süreli access, uzun süreli refresh)
- Kurumsal entegrasyonlar için API key'ler
- Admin/finans hesapları için opsiyonel 2FA
- Login denemelerinde rate limiting
- Parola politikaları zorunlu

## Sevk & Rotalama

**Atama Stratejileri:**
- **Manuel** - MVP için birincil (operasyon ekibi atar)
- **Otomatik kural tabanlı** - Yakınlık, kurye müsaitliği, hizmet tipi önceliği
- **Gelişmiş (gelecek)** - Multi-drop optimizasyonu, TSP heuristikleri

**Rotalama Entegrasyonu:**
- Google Maps API / OSRM / GraphHopper rota optimizasyonu için

## Entegrasyonlar

**SMS Gateway** - Netgsm, Turkcell API, Twilio
**Ödeme** - iyzico, PayU, Param (Türkiye odaklı)
**E-fatura** - GİB e-ARŞİV/e-FATURA entegrasyonu
**Haritalar** - Google Maps veya OpenStreetMap (Leaflet + OSRM)

## Uyumluluk & Güvenlik

**KVKK (Türk GDPR'ı):**
- Onay ile kişisel veri işleme
- Veri saklama politikaları (muhasebe için 7 yıl)
- Silme hakkı süreçleri
- Tüm veri erişimleri için audit log'ları

**Şifreleme:**
- Her yerde TLS (HTTPS)
- Gerekirse hassas veriler için alan düzeyinde şifreleme
- Kart saklama yok (ödeme sağlayıcı tokenizasyonu kullan)

**Dosya Depolama:**
- Fotoğraflar, imzalar, dokümanlar → S3/MinIO (DB'de değil)
- DB sadece URL/referans saklar

## Altyapı

**MVP Kurulumu:**
- Tek VPS üzerinde Docker Compose
- PostgreSQL + Redis + MinIO
- CDN ile statik frontend

**Production:**
- Kubernetes (yönetilen veya self-hosted)
- Read replica'lı yönetilen PostgreSQL
- Yönetilen Redis
- Message broker (Kafka/RabbitMQ/Redis Streams)
- S3-uyumlu object storage
- CDN (Cloudflare)

**İzleme:**
- Prometheus + Grafana (metrikler)
- ELK/Loki (loglama)
- Jaeger/OpenTelemetry (trace'ler)
- Alertmanager → Slack/PagerDuty

## Önemli Performans Hususları

- Zaman serisi verileri (events) partitioning/arşivleme gerektirir
- Rate card'ları ve bölge aramalarını Redis'te cache'le
- Fotoğraf, e-posta, SMS, fatura üretimi için async worker'lar
- Doğru indekslerle sorgu optimizasyonu

## Geliştirme Fazları

**Faz 1 - MVP (6-8 hafta):**
- Temel frontend (müşteri + kurumsal)
- Backend CRUD, admin paneli, kurye mobil web
- SMS/e-posta bildirimleri
- Manuel + kural tabanlı sevk

**Faz 2 - Operasyonlar & Ölçek (4-6 hafta):**
- Rota optimizasyonu
- CSV toplu yükleme
- Faturalama modülü
- Raporlar

**Faz 3 - Olgunlaşma (4-8 hafta):**
- Canlı GPS takibi
- İmza/fotoğraf iş akışları
- E-fatura entegrasyonu
- Native mobil uygulamalar

## Önemli Notlar

- Hızlı iterasyon için modüler monolit ile başla
- Webhook'lar kurumsal müşteriler için kritik
- CSV toplu işleme MVP'de kurumsal müşteriler için olmalı
- Tüm durum geçişleri shipment_events'e event-sourced olmalı
- Binary dosyalar (fotoğraf, imza) object storage'a gider, asla DB'ye değil
- KVKK uyumluluğu baştan tasarlanmalı

---
> Source: [byhawk/kuryem](https://github.com/byhawk/kuryem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
