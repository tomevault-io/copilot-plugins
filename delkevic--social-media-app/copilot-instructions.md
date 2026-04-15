## social-media-app

> - JavaScript (Frontend)

# ==========================
# 🔥 Cursor Project Rules 🔥
# ==========================

# 🔹 Genel Proje Kuralları
genel:
  dil: Türkçe
  tema: Koyu
  kodlama_dilleri:
    - JavaScript (Frontend)
    - Go (Backend)

# 🔹 Kod Stili Kuralları
kod_stili:
  girinti:
    frontend: 2 boşluk
    backend: tab
  maksimum_satır_uzunluğu: 100
  noktalı_virgül_kullanımı: true # Frontend için geçerli
  tırnak_stili: "'" # Tek tırnak kullanımı (Frontend)
  yorum_dili: Türkçe

# 🔹 İsimlendirme Standartları
isimlendirme:
  değişkenler: camelCase
  fonksiyonlar:
    frontend: camelCase
    backend: PascalCase # Go standartlarına uygun
  komponentler: PascalCase
  sabitler: UPPERCASE_SNAKE_CASE

# 🔹 Dosya ve Dizin Yapısı
dosya_yapısı:
  komponentler: PascalCase.jsx
  util_dosyaları: camelCase.js/.go
  backend_controller_dosyaları: camelCaseController.go

# 🔹 Import Kuralları
import_kuralları:
  sıralama:
    1: Standart Kütüphaneler (React/Go)
    2: Harici Kütüphaneler (Lodash, Axios vb.)
    3: Proje içi modüller (Komponentler, Context, Services)
    4: Stil dosyaları
  gruplar_arası_boşluk: true

# 🔹 Frontend (React) Komponent Yapısı
frontend_komponent_yapısı:
  yapı_sırası:
    1: State ve Hooklar
    2: Yardımcı fonksiyonlar
    3: useEffect kullanımı
    4: JSX render
  props_sırası:
    1: id
    2: className
    3: style
    4: diğerleri
    5: event handler'lar
  destructuring_kullanımı: true

# 🔹 Stil Kuralları (Frontend)
stil_kuralları:
  css_framework: Tailwind CSS
  inline_stil_kullanımı: false # Login.jsx istisna
  global_stil_dosyası: styles.css

# 🔹 Backend (Go) Yapısı ve Kuralları
backend_yapısı:
  proje_mimarisi: Controller-Route-Model
  veritabanı_kütüphanesi: GORM
  hata_yönetimi:
    - c.JSON ile hata response dön
    - Panik (panic()) yerine error handling kullan
  endpoint_standartları:
    - Versiyonlama kullan (/api/v1/products)
    - GET isteklerinde gereksiz büyük response'ları önle

# 🔹 Yetkilendirme ve Güvenlik Standartları
yetkilendirme:
  frontend:
    - Kullanıcı oturumunu context veya Redux ile yönet
    - Token saklama: HTTPOnly Cookie tercih edilir
  backend:
    - JWT veya OAuth2 kullanımı zorunlu
    - Yetkisiz isteklerde 401 Unauthorized dön
    - Admin erişimleri middleware içinde belirlenmeli

# 🔹 Uyarılar ve İyi Uygulamalar
uyarılar:
  - console.log kullanımı kontrol edilmeli
  - Kullanılmayan değişkenler temizlenmeli
  - Dosya yapısına sadık kalınmalı
  - API çağrıları merkezi bir dosyadan yönetilmeli

tasarımlar için material ui gibi küthüphaneler kullanılabilir

söylemediğim yerleri kafana göre değiştirme sadece söylediklerimi yap

gitignore dosyasına eklenmesi gereken bir şey olursa .gitignore dosyasına ekle

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Delkevic) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
