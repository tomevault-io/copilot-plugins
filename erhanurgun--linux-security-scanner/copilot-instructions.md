## linux-security-scanner

> - **Proje:** Ubuntu Security Scanner v0.3.5

# Ubuntu Security Scanner - Geliştirme Kuralları

## Proje Bilgisi
- **Proje:** Ubuntu Security Scanner v0.3.5
- **Dil:** Bash (shell scripting)
- **Amaç:** Ubuntu sistemler için güvenlik tarama aracı

## Versiyonlama (SemVer)

**Her değişiklikte versiyon güncellemesi ZORUNLUDUR!**

Format: `MAJOR.MINOR.PATCH`

| Değişiklik Türü | Versiyon | Örnek |
|-----------------|----------|-------|
| Breaking change (API değişikliği, eski uyumluluk kırılması) | MAJOR | 0.2.0 -> 1.0.0 |
| Yeni özellik (geriye uyumlu) | MINOR | 0.2.0 -> 0.3.0 |
| Bug fix, küçük düzeltme | PATCH | 0.2.0 -> 0.2.1 |

### Güncellenecek Dosyalar
Versiyon değiştiğinde şu dosyalar güncellenmelidir:
1. `bin/security-scanner` - `SCANNER_VERSION` değişkeni
2. `CLAUDE.md` - Proje bilgisi bölümü
3. `CHANGELOG.md` - Değişiklik kaydı

### Değişiklik Kaydı
- **v0.3.5** - Alert-remediation tam eşleştirme (63 yeni entry, 2 yeni JSON dosyası)
- **v0.3.4** - Remediation sistemi tam düzeltme (~79 yeni entry, 6 kategori eşleştirmesi)
- **v0.3.3** - Bug fix: CRLF line endings (lib/reporting/), wc -l output normalization (05-network.sh)
- **v0.3.2** - Final refactoring: Config birleştirme, eksik dosyalar, dokümantasyon
- **v0.3.0** - SRP modül bölme (lib/reporting/ dizini), reporting.sh wrapper olarak güncellendi
- **v0.2.2** - Türkçe karakter düzeltmeleri, DRY helper fonksiyonlar
- **v0.2.1** - Tab active state bug fix, Tümü Aç/Kapat kaldırıldı, Her satıra Detay butonu eklendi
- **v0.2.0** - Modal Dialog eklendi, Tab active state ve hidden/show bug fix'leri
- **v0.1.0** - İlk sürüm, TXT kaldırıldı, JSON/HTML modernize edildi

## Kodlama Kuralları

### Genel
- Türkçe karakterler (ç, ğ, ı, ö, ş, ü, Ç, Ğ, İ, Ö, Ş, Ü) her zaman UTF-8 olarak yazılacak
- Hiçbir şekilde emoji kullanılmayacak
- HTML entity emoji formatları (`&#xxxxx;`) yasak
- Gerekli görsel öğeler için SVG ikon seti (Heroicons) kullanılacak

### HTML/JavaScript
- innerHTML yerine textContent ve güvenli DOM API'leri kullanılacak (XSS önlemi)
- TailwindCSS utility class'ları tercih edilecek
- Inline CSS yerine class'lar kullanılacak

### Bash
- `set -uo pipefail` her zaman kullanılacak
- Değişkenler `${VAR}` formatında yazılacak
- Fonksiyon isimleri snake_case olacak

## Dizin Yapısı
```
├── bin/                 # Ana çalıştırılabilir dosya
├── lib/                 # Kütüphane dosyaları
│   ├── reporting/       # Raporlama alt modülleri (SRP uyumu)
├── modules/             # Tarama modülleri
├── templates/           # CSS ve JS şablonları
│   ├── css/
│   └── js/
├── config/              # Yapılandırma dosyaları
├── data/                # Veri dosyaları
└── logs/                # Log dosyaları
```

## Raporlama
- Desteklenen formatlar: terminal, json, html
- TXT format kaldırıldı
- JSON yapısı: meta, scan, host, summary, findings_by_category, findings
- HTML: TailwindCSS ile modern tasarım

## Geliştirme Yaklaşımı

### Dil ve İletişim
- Türkçe iletişim (Türkçe karakterler olduğu gibi: ç, ğ, ı, ö, ş, ü)
- Yorum satırları Türkçe, kod (değişken, fonksiyon) İngilizce
- Kısa ve öz yorum satırları (gereksiz açıklamalardan kaçın)

### Yazılım Prensipleri (Zorunlu)
- **SOLID** - Single Responsibility, Open/Closed, Liskov, Interface Segregation, Dependency Inversion
- **DRY** - Don't Repeat Yourself
- **KISS** - Keep It Simple, Stupid
- **YAGNI** - You Aren't Gonna Need It

### Karar Alma Süreci
- Değişiklik öncesi analiz yap ve en iyi seçeneği öner
- Alternatif yaklaşımları listele (tercih edilen 1. sırada)
- Seçenekleri terminal'den seçilebilir formatta sun (AskUserQuestion)
- Yanlış yönlendirmelerde uyar ve doğru yaklaşımı öner
- Kabul etmeden önce sorgula, Sr seviyesinde davran

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/erhanurgun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
