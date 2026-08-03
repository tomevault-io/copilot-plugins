## controller-rules

> Controller'lar mimarinin giriş noktasıdır ve mümkün olduğunca "zayıf" (thin) tutulmalıdır. Sorumlulukları kesin olarak belirlenmiştir.

# Controller Katmanı Kuralları

Controller'lar mimarinin giriş noktasıdır ve mümkün olduğunca "zayıf" (thin) tutulmalıdır. Sorumlulukları kesin olarak belirlenmiştir.

## Controller'ların Görevleri
Bir controller metodunun yapması gerekenler sadece şunlardır:
1.  Gelen HTTP isteğini karşılamak.
2.  İlgili `FormRequest` sınıfını kullanarak isteğin validasyonunu ve yetkilendirmesini tetiklemek.
3.  Doğrulanmış veriyi bir `DTO` (Data Transfer Object) nesnesine dönüştürmek.
4.  Bu DTO'yu ilgili `Service` metoduna parametre olarak geçmek.
5.  Servisten dönen `ServiceResponse` nesnesini doğrudan `return` etmek.

## Yasaklar
- **İş Mantığı YOK**: Controller içinde `if/else`, `foreach` gibi kontrol yapıları veya herhangi bir iş mantığı bulunmamalıdır.
- **Doğrudan Veritabanı Erişimi YOK**: Controller'lar asla `Eloquent` modellerini veya `Repository`'leri doğrudan çağırmamalıdır.
- **Manuel Validasyon YOK**: `$request->validate()` gibi manuel validasyon yöntemleri kullanılmamalıdır. Her zaman FormRequest kullanılmalıdır.
- **Doğrudan Yanıt Oluşturma YOK**: `response()->json(...)` gibi manuel yanıtlar oluşturulmamalıdır. Servisten dönen `ServiceResponse` yeterlidir.
- **Model Binding YOK**: Bu projede model binding kullanmıyoruz. Controller method imzalarında model parametreleri (örn: `Survey $survey`) bulunmamalıdır. ID'ler request'ten alınmalıdır.

## Örnek Yapı

```php
class SurveyController extends Controller
{
    public function __construct(
        private readonly SurveyServiceInterface $surveyService
    ) {}

    public function store(CreateSurveyRequest $request): ServiceResponse
    {
        $dto = new SurveyDto(...$request->validated());
        return $this->surveyService->create($dto);
    }
    
    // Model binding kullanmıyoruz - ID'yi request'ten alıyoruz
    public function show(Request $request, int $id): ServiceResponse
    {
        return $this->surveyService->getById($id);
    }
}
```

---
> Source: [fzengin19/polling](https://github.com/fzengin19/polling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
