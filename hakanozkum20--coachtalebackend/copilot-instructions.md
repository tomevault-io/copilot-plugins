## coachtalebackend

> - Proje Clean Architecture ve SOLID prensiplerine uygun olarak geliştirilecektir.

# Cursor Rules: Clean Architecture & SOLID Kuralları

## Genel Mimari
- Proje Clean Architecture ve SOLID prensiplerine uygun olarak geliştirilecektir.
- Katmanlar arası bağımlılık en aza indirilecek, her katman sadece kendi sorumluluğunda olan işleri yapacaktır.

## Klasör Yapısı ve Katmanlar

- **API.Core**
  - Domain/Entities: Temel iş nesneleri (ör: Product, User)
  - BaseEntity: Tüm entity’lerin ortak özellikleri (Id, CreatedDate, UpdatedDate, vs.)

- **Application**
  - Abstractions: Soyutlamalar (ör: IRepository, IReadRepository, IWriteRepository)
  - Services: Domain nesneleriyle ilgili iş mantığı (ör: ProductService)
  - Mappers: Entity <-> DTO dönüşümleri

- **API.Infrastructure**
  - Infrastructure: Servis kayıtları, ayarlar, validation filtreleri, enums
  - Persistence: Veritabanı context’leri, migration’lar, repository’lerin somut halleri (ör: ProductRepository)

- **API.Presentation**
  - Controllers: HTTP endpoint’leri, route’lar
  - DTOs: API’ye özel veri transfer nesneleri

## Temel Kurallar
- DTO’lar sadece Presentation katmanında tutulacak.
- Mapper’lar Application katmanında olacak.
- Domain katmanı dış katmanlara bağımlı olmayacak.
- Soyutlamalar (interface/abstraction) Application katmanında olacak, somutlar Infrastructure’da.
- Validation, logging, error handling gibi cross-cutting concerns Infrastructure’da yönetilecek.
- Dependency Injection (DI) ile servisler loosely coupled olacak.

## Gelecekte Eklenebilecekler
- **Exception Handling:** API.Presentation’da global error handler veya middleware eklenebilir.
- **Validation:** API.Infrastructure’da validation filtreleri/middleware’leri eklenebilir.
- **Logging:** API.Infrastructure’da merkezi bir logger servisi eklenebilir.
- **Config Management:** API.Infrastructure’da config klasörü veya .env dosyası ile ortam değişkenleri yönetilebilir.
- **Unit Test:** Application ve Core katmanları için test altyapısı kurulabilir.
- **Authorization/Authentication:** Gerekirse API.Infrastructure’da auth servisleri/middleware’leri eklenebilir.
- **Mapping:** Entity <-> DTO dönüşümleri için otomatik veya manuel yardımcılar eklenebilir.

## Notlar
- CQRS ve MediatR kullanılmayacak.
- SOLID prensiplerine (SRP, OCP, LSP, ISP, DIP) uyulacak.
- Katmanlar arası bağımlılık sadece yukarıdan aşağıya olacak (örn: Presentation → Application → Core).

---
Bu dosya, proje geliştirme sürecinde referans olarak kullanılacaktır. Yeni kurallar veya değişiklikler buraya eklenmelidir. 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hakanozkum20) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
