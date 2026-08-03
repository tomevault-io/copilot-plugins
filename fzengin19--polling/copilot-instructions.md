## repository-rules

> Repository katmanı, veritabanı ile olan tüm iletişimi soyutlar. Uygulamanın geri kalanı, verinin nasıl saklandığını veya getirildiğini bilmek zorunda değildir.

# Repository Katmanı Kuralları

Repository katmanı, veritabanı ile olan tüm iletişimi soyutlar. Uygulamanın geri kalanı, verinin nasıl saklandığını veya getirildiğini bilmek zorunda değildir.

## Temel Sorumluluklar
- **Veri Erişimi**: `Eloquent` modellerini kullanarak veritabanından veri okuma (SELECT) ve yazma (CREATE, UPDATE, DELETE) işlemlerini gerçekleştirmek.
- **Soyutlama**: Karmaşık `Eloquent` sorgularını, `join`'leri veya özel mantıkları basit ve anlaşılır metot isimleri arkasına gizlemek.
- **Arayüz (Interface) Implementasyonu**: Her somut repository (`Eloquent`), `app/Repositories/Abstract` altında tanımlanmış olan kendi arayüzünü (`interface`) implement etmelidir.

## Kurallar
1.  **Sadece Veritabanı**: Repository'ler sadece ve sadece `Eloquent` modelleri ile konuşmalıdır. İçlerinde kesinlikle iş mantığı, HTTP bilgisi veya başka bir katmana ait kod bulunmamalıdır.
2.  **İş Mantığı YOK**: Bir kullanıcının bir anketi düzenleyip düzenleyemeyeceğini kontrol etmek gibi mantıklar burada yer almaz. Repository sadece "anketi ID ile bul" veya "kullanıcıyı e-posta ile bul" gibi net veri operasyonları yapmalıdır.
3.  **Dönüş Tipleri**: Repository metotları genellikle `Eloquent Model`, `Collection`, `Paginator` nesneleri, `boolean` veya `null` gibi ham veritabanı sonuçlarını dönmelidir. `ServiceResponse` veya `DTO` gibi üst katman yapılarını bilmez ve kullanmaz.
4.  **İsimlendirme**: Metot isimleri, yaptıkları işi açıkça belirtmelidir. Örneğin, `find(int $id)`, `getByStatus(string $status)`, `create(array $data)`, `update(int $id, array $data)`.

## Örnek Yapı
```php
class UserRepository extends BaseRepository implements UserRepositoryInterface
{
    public function __g(User $model)
    {
        parent::__construct($model);
    }

    public function findByEmail(string $email): ?User
    {
        return $this->model->where('email', $email)->first();
    }
}
```
*Not: Örnekteki `BaseRepository`, sık kullanılan `all()`, `find()`, `create()`, `update()`, `delete()` gibi temel CRUD metotlarını içererek kod tekrarını azaltır.*

---
> Source: [fzengin19/polling](https://github.com/fzengin19/polling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
