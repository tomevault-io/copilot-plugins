## php-laravel-rules

> Bu dosya, projedeki tüm PHP ve Laravel kodları için geçerli olan stil, yapı ve en iyi pratikleri içerir.

# PHP ve Laravel Geliştirme Kuralları

Bu dosya, projedeki tüm PHP ve Laravel kodları için geçerli olan stil, yapı ve en iyi pratikleri içerir.

## Temel İlkeler
- **SOLID**: Kodunuzu SOLID prensiplerine uygun yazın. Her sınıfın ve metodun tek bir sorumluluğu olmalıdır.
- **Strict Types**: Tüm PHP dosyalarında `declare(strict_types=1);` kullanın.
- **Interface Kullanımı**: Servisler ve Repository'ler daima bir arayüz (`interface`) üzerinden kullanılmalıdır. Bu, bağımlılıkları esnek ve test edilebilir kılar.
- **Dependency Injection (DI)**: Bağımlılıkları (örneğin bir servise repository enjekte etmek) daima sınıfın constructor'ı üzerinden `public readonly` property'ler kullanarak enjekte edin ve arayüzleri type-hint olarak kullanın.

## Laravel'e Özgü Kurallar
1.  **FormRequest**: Validasyon ve yetkilendirme (authorization) mantığı **sadece** `App\Http\Requests` altındaki FormRequest sınıflarında bulunmalıdır. Controller içinde `$request->validate()` gibi manuel validasyon yapılmamalıdır.
2.  **DTO (Data Transfer Object)**: Controller'da, FormRequest'ten gelen doğrulanmış veriyi Service katmanına aktarmak için **her zaman** bir DTO (`App\Dtos`) kullanılmalıdır.
3.  **Service Katmanı**: Tüm iş mantığı `App\Services\Concrete` içindeki servis sınıflarında yer almalıdır. Servisler, kendilerine ait arayüzleri (`App\Services\Abstract`) implement etmelidir.
4.  **Repository Katmanı**: Tüm veritabanı işlemleri `App\Repositories\Eloquent` içindeki repository sınıflarında soyutlanmalıdır. Repository'ler, kendilerine ait arayüzleri (`App\Repositories\Abstract`) implement etmelidir. Servisler veya Controller'lar asla doğrudan Eloquent modeli kullanmamalıdır.
5.  **API Resources**: API yanıtlarını formatlamak için **her zaman** `App\Http\Resources` altındaki resource sınıfları kullanılmalıdır. Controller'dan veya servislerden doğrudan Eloquent modeli veya generic array dönülmemelidir.
6.  **ServiceResponse**: Tüm servis metotları, `App\Responses\ServiceResponse` sınıfından bir nesne dönmelidir. Bu, API yanıtlarında tutarlılık sağlar.
7.  **Provider'lar**: Yeni bir Service veya Repository oluşturduğunuzda, ilgili arayüz-sınıf eşleşmesini `App\Providers` altındaki uygun bir provider'a (`DomainServiceProvider` veya `RepositoryServiceProvider`) kaydetmeyi unutmayın.
8.  **Model İlişkileri**: Veritabanı model yapısı ve ilişkiler için `PROJECT_BASE_STRUCTURE.md` dosyası referans alınmalıdır.

---
> Source: [fzengin19/polling](https://github.com/fzengin19/polling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
