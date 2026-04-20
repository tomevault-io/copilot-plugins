## kocmax

> **İş Sağlığı Güvenliği Kıyafetleri E-Ticaret Platformu** üzerinde çalışıyorsunuz - **Faz 1** geliştirme aşamasındaki B2B/B2C hibrit e-ticaret platformu.


# OTOMATİK YÜKENEN PROJE CONTEXT'İ

## 🎯 GÜNCEL PROJE DURUMU
**İş Sağlığı Güvenliği Kıyafetleri E-Ticaret Platformu** üzerinde çalışıyorsunuz - **Faz 1** geliştirme aşamasındaki B2B/B2C hibrit e-ticaret platformu.

## 🛠️ TEKNOLOJİ STACK
- **Backend**: Laravel 11 + PHP 8.2 + Filament 3
- **Veritabanı**: MySQL 8.0 (Docker)
- **Frontend**: React 18 + TypeScript + shadcn/ui (planlanmış)
- **Yetkilendirme**: Spatie Laravel Permission
- **Özellikler**: Çoklu para birimi, bayi yönetimi, kompleks ürün varyantları

## 📋 HER YANITTAN ÖNCE ZORUNLU OKUMA
1. **`memory.md`** - Mevcut proje durumu ve genel bakış
2. **`memorybank/development-rules.md`** - Kodlama standartları (PSR-12, strict typing)
3. **`memorybank/common-patterns.md`** - Implementasyon pattern'leri ve örnekler

## ⚡ HIZLI REFERANS
- **Model'ler**: PascalCase tekil (Product, DealerApplication)
- **Tablolar**: snake_case çoğul (products, dealer_applications)
- **Para**: Her zaman decimal(10,2)
- **Foreign Key'ler**: {table}_id formatı
- **Servisler**: Business logic katmanı (ExchangeRateService, PricingService)
- **Politikalar**: Yetkilendirme (ProductPolicy, UserPolicy)

## 🔒 GÜVENLİK KONTROL LİSTESİ
- [ ] Tüm kullanıcı girişlerini validate et
- [ ] Yetkilendirme için policy'leri kullan
- [ ] Hata yönetimi ekle
- [ ] Strict typing kullan
- [ ] Service layer pattern'ini takip et

## 📁 TEMEL DİZİNLER
```
app/
├── Models/          # Eloquent modeller
├── Services/        # Business logic
├── Filament/        # Admin paneli
├── Policies/        # Yetkilendirme
└── Http/            # Controller'lar, Request'ler
```

## 🎯 GÜNCEL FAZ ÖZELLİKLERİ
✅ Kullanıcı yönetimi + bayi başvuruları
✅ Varyantlı ürün kataloğu
✅ Para birimi + döviz kurları (TCMB)
✅ Filament admin paneli
✅ Yetkilendirme politikaları

## 🔄 PROJE DURUMU
- **Tamamlanan**: Çekirdek altyapı, model'ler, basic CRUD
- **Devam Eden**: Dealer application sistemi
- **Bekleyen**: Gelişmiş ürün yönetimi, frontend, ödeme entegrasyonu

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oguzhanfiliz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
