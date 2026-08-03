## workflow

> Bu proje için geliştirme yaparken aşağıdaki iş akışı katı bir şekilde takip edilmelidir. Bu, projenin kararlılığını korumak ve regresyonları önlemek için kritiktir.

# Geliştirme ve Hata Düzeltme İş Akışı

Bu proje için geliştirme yaparken aşağıdaki iş akışı katı bir şekilde takip edilmelidir. Bu, projenin kararlılığını korumak ve regresyonları önlemek için kritiktir.

## 1. Geliştirme Döngüsü

1.  **Görev Belirleme (`roadmap.md`):** Her yeni geliştirme, `@roadmap.md` dosyasındaki bir adıma karşılık gelmelidir.
2.  **Planlama (`active_development_plan.md`):**
    *   `@roadmap.md`'den alınan görevin detaylı teknik planı `@active_development_plan.md` dosyasına yazılır.
    *   Plan, görevi tamamlamak için gereken tüm adımları (repository, servis, controller, test güncellemeleri vb.) içermelidir.
3.  **Geliştirme:**
    *   Geliştirme, `@active_development_plan.md` dosyasındaki adımlar takip edilerek yapılır.
    *   **Tek Adım Prensibi:** Her seferinde plandaki **sadece bir madde** hayata geçirilir.
    *   **Minimal Değişiklik Prensibi:** Bir adımı tamamlamak için gereken **en küçük ve en odaklı değişiklik** yapılmalıdır.

## 2. Test ve Doğrulama Döngüsü

1.  **Test Çalıştırma:** Plan dosyasındaki bir adım tamamlandıktan sonra, projenin tüm test paketi çalıştırılır.
    *   **Komut:** `php artisan test`
2.  **Sonuç Değerlendirme:**
    *   **Tüm Testler Başarılıysa:**
        *   `@active_development_plan.md` dosyasında ilgili madde işaretlenir (`[x]`).
        *   Plandaki bir sonraki adıma geçilir.
        *   Eğer plandaki tüm adımlar tamamlandıysa, `@active_development_plan.md`'deki ilgili görev bölümü silinir ve `@roadmap.md`'de ana görev işaretlenir.
    *   **Başarısız Test Varsa:**
        *   Yeni bir geliştirme adımına **geçilmez**.
        *   Başarısız olan testlerin sonuçları analiz edilir ve `@errors.md` dosyasına yazılır.
        *   Hata, bir sonraki adım olarak `@active_development_plan.md`'e "HATA DÜZELTME:" ön ekiyle eklenir ve çözülene kadar geliştirme döngüsüne bu hatadan devam edilir.

---
> Source: [fzengin19/polling](https://github.com/fzengin19/polling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-03 -->
