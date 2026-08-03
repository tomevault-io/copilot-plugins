## model-rules

> Modeller, veritabanı tablolarının bir temsilidir ve sadece veri yapısıyla ilgili tanımlamaları içermelidir.

# Eloquent Model Kuralları

Modeller, veritabanı tablolarının bir temsilidir ve sadece veri yapısıyla ilgili tanımlamaları içermelidir.

## Sorumluluklar
- **Veritabanı Tablosunu Temsil Etme**: `protected $table`, `protected $primaryKey` gibi özelliklerle tabloyu tanımlamak.
- **Toplu Atama (Mass Assignment)**: `protected $fillable` veya `protected $guarded` dizilerini tanımlayarak güvenliği sağlamak.
- **Tip Dönüşümleri (Casts)**: `protected $casts` dizisi ile veritabanından gelen verilerin doğru tiplere (örn: `boolean`, `datetime`, `json`) dönüştürülmesini sağlamak.
- **İlişkiler (Relationships)**: `hasMany`, `belongsTo` gibi metotlarla diğer modellerle olan ilişkileri tanımlamak.
- **Tarih Alanları**: `protected $dates` veya `created_at`, `updated_at` gibi zaman damgalarını yönetmek.

## Yasaklar
- **İş Mantığı YOK**: Modellerin içine iş mantığı eklenmemelidir. Örneğin, bir şifrenin hash'lenmesi gibi işlemler `Observer`'lar veya `Service` katmanında yapılmalıdır. Bir model, bir anketin yayınlanıp yayınlanamayacağını bilmemelidir.
- **Karmaşık Sorgular YOK**: Karmaşık sorgu mantıkları model içinde scope olarak (`public function scopePublished(Builder $query)`) eklenebilir ancak bu sorguların çağrılması ve birleştirilmesi Repository katmanının sorumluluğundadır.
- **Doğrudan Kullanım YOK**: Uygulamanın geri kalanı (Controller, Service) modelleri doğrudan kullanmamalı, bunun yerine Repository katmanı üzerinden etkileşim kurmalıdır.

## Referans
- Tüm modellerin sütunları ve ilişkileri için projenin ana dökümanı olan `PROJECT_BASE_STRUCTURE.md` dosyası tek doğru kaynak olarak kabul edilmelidir.

---
> Source: [fzengin19/polling](https://github.com/fzengin19/polling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
