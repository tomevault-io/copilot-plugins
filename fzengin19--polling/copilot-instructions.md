## testing-rules

> Testler, uygulamanın doğruluğunu ve kararlılığını garanti altına almanın en önemli parçasıdır. Tüm yeni özellikler ve hata düzeltmeleri testlerle desteklenmelidir.

# Test Yazma Kuralları

Testler, uygulamanın doğruluğunu ve kararlılığını garanti altına almanın en önemli parçasıdır. Tüm yeni özellikler ve hata düzeltmeleri testlerle desteklenmelidir.

## Genel Kurallar
- **Pest Framework**: Tüm testler `Pest` test çatısı kullanılarak yazılmalıdır. `it()` ve `test()` fonksiyonları kullanılmalıdır.
- **Açıklayıcı İsimler**: Test isimleri, test edilen senaryoyu açık ve net bir şekilde anlatmalıdır. Örneğin, `it('cannot create a survey without authentication')`.
- **Veritabanı Temizliği**: Her testten sonra veritabanının temizlendiğinden emin olmak için `RefreshDatabase` trait'i kullanılmalıdır.
- **Factory Kullanımı**: Test verisi oluşturmak için **her zaman** `Eloquent Factory`'ler (`database/factories`) kullanılmalıdır.

## Feature Testleri (`tests/Feature`)
- **Odak**: Feature testleri, API endpoint'lerini dışarıdan bir kullanıcı gibi test etmelidir. Uygulamanın iç yapısını (servisler, repository'ler) bilmemelidir.
- **Akış**: Bir feature testi genellikle şu adımları izler:
    1.  Gerekli veriyi factory'ler ile oluştur (`User`, `Survey` vb.).
    2.  Gerekirse `actingAs($user)` ile kullanıcıyı authenticate et.
    3.  İlgili API endpoint'ine bir istek yap (`$this->getJson()`, `$this->postJson()` vb.).
    4.  Gelen yanıt üzerinde `assert`'ler yap.
- **Assertion'lar**: Yanıt üzerinde en azından şunlar kontrol edilmelidir:
    -   Doğru HTTP durum kodu (`assertStatus(200)`, `assertCreated()`, `assertNotFound()`, `assertForbidden()`).
    -   Yanıtın JSON yapısının beklenen API standardına uyması (`assertJsonStructure`, `assertJsonFragment`).
    -   Veritabanında beklenen değişikliğin gerçekleşmesi (`assertDatabaseHas`, `assertDatabaseMissing`).

## Unit Testleri (`tests/Unit`)
- **Odak**: Unit testleri, tek bir sınıfın veya metodun (genellikle `Service` veya karmaşık bir Action sınıfı) mantığını izole bir şekilde test etmelidir.
- **Mocking**: Bağımlılıkları (örneğin bir servisin bağımlı olduğu repository) mock'layarak testi izole edin.
- **Kullanım Alanı**: Daha çok karmaşık algoritmalar, hesaplamalar veya özel iş kurallarını test etmek için kullanılır. Projenin ana test yükü Feature testlerinde olmalıdır.

---
> Source: [fzengin19/polling](https://github.com/fzengin19/polling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
