## service-rules

> Service katmanı, uygulamanın tüm iş mantığının (business logic) bulunduğu yerdir.

# Service Katmanı Kuralları

Service katmanı, uygulamanın tüm iş mantığının (business logic) bulunduğu yerdir.

## Temel Sorumluluklar
- **İş Mantığı**: Bir özelliğin veya kullanım senaryosunun gerektirdiği tüm adımları, kontrolleri ve operasyonları yönetmek.
- **Orkestrasyon**: Gerekli `Repository` metotlarını çağırmak, diğer `Service`'lerle iletişim kurmak, event'leri tetiklemek gibi işlemleri koordine etmek.
- **Arayüz (Interface) Implementasyonu**: Her somut servis (`Concrete`), `app/Services/Abstract` altında tanımlanmış olan kendi arayüzünü (`interface`) implement etmelidir.
- **Bağımlılıklar**: Diğer katmanlara (çoğunlukla Repository'ler) olan bağımlılıklar, constructor üzerinden arayüzleri kullanılarak enjekte edilmelidir.

## Kurallar
1.  **Sadece İş Mantığı**: Bu katman, HTTP'ye özgü (`Request`, `Response`) hiçbir şey bilmemelidir. Sadece DTO'lar ve temel veri tipleri ile çalışır.
2.  **Repository Kullanımı**: Veritabanı işlemleri için daima Repository katmanını kullanmalıdır. Asla doğrudan `Eloquent` modeli (`User::create()`) kullanmamalıdır.
3.  **Standart Yanıt**: Tüm public metotlar, işlemin sonucunu (başarı, hata, veri) sarmalayan bir `App\Responses\ServiceResponse` nesnesi döndürmelidir.
4.  **Hata Yönetimi**: Beklenen iş mantığı hataları (örn: "Kullanıcı zaten mevcut") `ServiceResponse` içinde yönetilmelidir. Beklenmedik istisnalar (exception) için merkezi hata yöneticisi devreye girecektir, bu nedenle gereksiz `try-catch` bloklarından kaçınılmalıdır.

## Örnek Yapı

```php
class AuthService implements AuthServiceInterface
{
    public function __construct(
        private readonly UserRepositoryInterface $userRepository,
        private readonly ResourceMapInterface $resourceMap
    ) {}

    public function register(RegisterDto $dto): ServiceResponse
    {
        if ($this->userRepository->findByEmail($dto->email)) {
            return ServiceResponse::error('Bu e-posta adresi zaten kullanılıyor.');
        }

        $user = $this->userRepository->create([...]);
        $token = $user->createToken('api-token')->plainTextToken;

        $data = [
            'user' => $this->resourceMap->map($user),
            'token' => $token,
        ];
        
        return ServiceResponse::success('Kayıt başarılı.', $data, 201);
    }
}
```

---
> Source: [fzengin19/polling](https://github.com/fzengin19/polling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
