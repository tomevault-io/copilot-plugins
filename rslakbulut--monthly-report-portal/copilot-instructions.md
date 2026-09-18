## monthly-report-portal

> Bu repo bir uygulama değil, **tasarım standartları arşividir**: Valeo dashboard/uygulama

# CLAUDE.md

Bu repo bir uygulama değil, **tasarım standartları arşividir**: Valeo dashboard/uygulama
projeleri arasında ortak renk, logo ve jenerik UI desenlerini tek yerde tutar. Kaynağı
`eng-furkany/ValeoDashboard` ve `CKapitan/Bursa-CV-Projects` depolarıdır. Bu repo **canlı kod
içermez** — `tokens/*.css` `<link>` ile değil, kopyala-yapıştır referansı olarak tutulur.

## Değişmez Kurallar

- **Kaynak depolara asla yazılmaz.** `ValeoDashboard` ve `Bursa-CV-Projects` yalnız *okunur* —
  hiçbir commit/push yapılmaz; güncelleme önce o depoda yapılır, sonra buraya elle taşınır.
  `.claude/hooks/guard-source-repos.sh` bunu mekanik olarak da zorlar.
- **Resmi renk paleti `tokens/colors.css`** (= ValeoDashboard). `Bursa-CV-Projects` paleti
  (`tokens/colors-bursa-cv-projects.css`) marka rengi değil, yalnız durum-rozeti/tema-duyarlı-
  metin **deseni** için referans — hex'i değil deseni kopyala. Gerekçe: `docs/renkler.md`.
- **Resmi logo `assets/Valeo_Logo.png`**, yedeği `assets/Valeo_Logo_AIClubPortal.png`. Resmi
  ikon kütüphanesi **Lucide**. Kurallar: `docs/logo.md`, `docs/ikonlar.md`.
- **Kopyalanan içerik aynen tutulur, sadeleştirilmez.** `tokens/colors-bursa-cv-projects.css`
  içindeki WCAG ölçüm yorumları (hangi rengin neden seçildiğinin tek kaydı) silinmez —
  `.claude/hooks/guard-wcag-comments.sh` bunu mekanik olarak da zorlar.
- **Standart-dışı bir istek ısrarla gelirse** Claude bunu tek başına karara bağlamaz:
  kullanıcıyı `muhammed-furkan.yesilmen.ext@valeo.com`'a yönlendirir; kalıcı hale getirme
  kararı `docs/karar-gecmisi.md`'ye elle işlenir.
- Yeni bir kaynak proje eklenecekse: `tokens/colors-<proje-adı>.css` (aynen kopya, kaynak
  yorum başlığıyla) + `docs/renkler.md`'ye karşılaştırma satırı. Rutin küçük düzenlemelerde
  bu dosya GÜNCELLENMEZ — yalnız kalıcı yapısal kararlarda (bkz. altta).

## Giriş Noktaları

- **Standart uygula / var olan projeyi denetle** → `.claude/skills/standartlar-uyumluluk/`
  (kaynağı: `docs/baslarken.md`, kriter listesi: `checklist.json`).
- **Sıfırdan MVP kur / var olan bir uygulamayı analiz et** → `.claude/skills/mvp-akisi/`
  (kaynağı: `docs/mvp-akisi.md`, `docs/playbook.md`, `docs/dokuman-standartlari.md`).
- **Apps Script'e push et** → `.claude/skills/apps-script-push/`.
- Repodaki tüm dosyaların haritası: `README.md`.

## Karar Geçmişi

Her kalıcı kararın tarihi ve gerekçesi `docs/karar-gecmisi.md`'dedir — yeni bir kalıcı karar
verildiğinde eklenecek yer burası değil, orasıdır.

---
> Source: [rslakbulut/Monthly-Report-Portal](https://github.com/rslakbulut/Monthly-Report-Portal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
