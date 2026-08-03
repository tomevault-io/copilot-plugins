## general-architecture

> Bu proje, katmanlı ve SOLID prensiplerine uygun modern bir Laravel mimarisi kullanılarak geliştirilmiştir. Geliştirme yaparken aşağıdaki temel ilkelere ve yapılara her zaman uyulmalıdır. Referans döküman `docs/architecture.md` dosyasıdır.

# Proje Mimarisi ve Geliştirme Standartları

Bu proje, katmanlı ve SOLID prensiplerine uygun modern bir Laravel mimarisi kullanılarak geliştirilmiştir. Geliştirme yaparken aşağıdaki temel ilkelere ve yapılara her zaman uyulmalıdır. Referans döküman `docs/architecture.md` dosyasıdır.

## 1. Katmanlı Mimari

Proje aşağıdaki ana katmanlardan oluşur. Her katmanın net bir sorumluluğu vardır:

- **Controller**: HTTP isteklerini karşılar, `FormRequest` ile validasyon yapar, veriyi `DTO`'ya dönüştürür ve `Service` katmanına iletir. **İş mantığı içermez.**
- **Service**: Tüm iş mantığının bulunduğu yerdir. `Repository` ve diğer servislerle konuşur.
- **Repository**: Veritabanı işlemlerini soyutlar. Sadece `Eloquent` modelleri ile iletişim kurar.
- **DTO (Data Transfer Object)**: `Controller` ve `Service` katmanları arasında tip güvenli veri taşımak için kullanılır.
- **Resource**: API yanıtlarını standart bir formata dönüştürür.
- **Model**: Veritabanı tablolarını temsil eden `Eloquent` sınıflarıdır.

## 2. API İsteği Akışı

Bir API isteğinin yaşam döngüsü aşağıdaki gibidir:

`Route` -> `Controller` -> `FormRequest` (Validasyon) -> `DTO` -> `Service` (İş Mantığı) -> `Repository` -> `Model` -> `ServiceResponse` -> `Resource` -> `JSON Response`

## 3. API Yanıt Standardı (JSend Benzeri)

Tüm API yanıtları, frontend ile tutarlılığı sağlamak için aşağıdaki standart yapıyı takip etmelidir:

```json
{
    "success": true, // veya false
    "data": { ... } // veya [...] veya null
    "message": "İşlem başarılı." // veya "Hata mesajı."
}
```
- Servis katmanından dönen `ServiceResponse` nesnesi, bu yapıyı otomatik olarak oluşturur.

## 4. Ana Teknolojiler

- **Backend**: Laravel, PHP
- **Kimlik Doğrulama**: Laravel Sanctum (Token-based)
- **Rol ve Yetkilendirme**: `spatie/laravel-permission`
- **Medya Yönetimi**: `spatie/laravel-medialibrary`
- **Veritabanı Yapısı**: `PROJECT_BASE_STRUCTURE.md` dosyasında detaylandırılmıştır.
- **API Özellikleri**: `PROJECT_FEATURES.md` dosyasında özetlenmiştir.

---
> Source: [fzengin19/polling](https://github.com/fzengin19/polling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
