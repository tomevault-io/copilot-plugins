## autotts

> Ürün amacı, kabul kriterleri ve nihai karar insana aittir. AI analiz, öneri,

# AutoTTS katkı ve ajan sözleşmesi

Ürün amacı, kabul kriterleri ve nihai karar insana aittir. AI analiz, öneri,
kod ve test taslağı üretir. Açık insan seçimi bulunmayan bir öneriyi insan
kararı diye kaydetme. Kullanıcının bu görevde verdiği yetkiyi yeniden sorma.

## Görev başı

1. `git status --short` çalıştır; kullanıcı değişikliklerini koru. Depo yoksa
   bunu kaydet, branch/commit uydurma.
2. Etkilenen kodu, domain sözleşmelerini, `docs/ARCHITECTURE.md` ve ilgili
   `docs/decisions/` kayıtlarını oku.
3. Yetki kapsamını ve başlangıç kanıtını belirle. Ölçülmeyen baseline üretme.

## Uygulama

- GUI yalnız ana Tk iş parçacığında çalışır; model tek işçide kullanılır.
- Motor GUI'den bağımsızdır. Başarısız MPS denemesi kısmi çıktı yayımlamaz.
- Kullanıcı metni, tokenlar, credential dosyaları, cihaz UUID'si, gereksiz
  kişisel/finansal veri ve mutlak kullanıcı yolları kanıt belgelerine konmaz.
- Kullanıcı ekran görüntülerini depoya kopyalama. Cihaz bilgisini anonimleştir.
- AI'ya aktarılan veri türlerini belirt; tez artefaktlarının paylaşım iznini
  ayrıca doğrula. Dış servislere mesaj gönderme veya sürüm yayımlama için açık yetki gerekir.

## Görev sonu

Mimari/domain kararı oluştuysa (küçük UI rötuşları için zorunlu değildir):

1. `docs/evidence/TRACEABILITY.md` dosyasına gereksinim–karar–kod–test–sonuç
   bağlantısını yeni kayıt veya tarihli revizyon olarak ekle.
2. `docs/evidence/AI_COLLABORATION_LOG.md` indeksine ve ayrıntılarına oturum ekle;
   insan kararı ile AI önerisini ayır, düzeltilen/reddedilen AI çıktılarını kaydet.
3. Uygulandı, otomatik test geçti, hedef cihazda doğrulandı ve yayımlandı
   durumlarını ayrı yaz. Test, cihaz ve insan kabulünü birbirinden türetme.
4. Çok bileşeni etkileyen ve geri alınması yüksek regresyon riski taşıyan
   kararda ADR yaz. Kalıcı katkı kuralı değiştiyse bu sözleşmeyi de güncelle.
5. Sonradan yazılan kayıt `retrospective` olur. Bilinmeyen tarih, commit veya
   model sürümü için `kaydedilmedi` yaz.

## Belge değişmezliği

Mimari/geliştirme/kalite rehberleri yaşayan belgelerdir. Kanıt kayıtları
eklemeli tutulur; eski satırları sessizce değiştirme. Düzeltmede tarihli
revizyon ekle ve eski kaydı bağla. Eski ADR ancak açık durum değişikliği ve
tarihli açıklama ile işaretlenir; yeni karar ayrı ADR olur, iki yönlü bağlanır.

## Commit disiplini

- Her mantıksal birim ayrı commit; kanıt/doküman ayrı `docs:` commit.
- `git add <açık dosya listesi>` kullan; `git add -A` ve `git add .` kullanma.
- Commit öncesi `git diff --cached --name-only` ile kapsamı kontrol et.
- Commit kimliği bulunmadığında E1/E2 asgari kanıtı tamamlanmış gibi sunma.

---
> Source: [RuslanAeff/AutoTTS](https://github.com/RuslanAeff/AutoTTS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
