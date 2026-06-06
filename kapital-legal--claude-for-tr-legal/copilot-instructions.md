## claude-for-tr-legal

> Bu dosya, repo üzerinde çalışan geliştiriciler (insan veya AI) için yönlendirme verir. Plugin kullanıcıları için ana doküman [README.md](README.md)'dir.

# Claude for TR Legal — Geliştirici Notları

Bu dosya, repo üzerinde çalışan geliştiriciler (insan veya AI) için yönlendirme verir. Plugin kullanıcıları için ana doküman [README.md](README.md)'dir.

> **Türev Eser Notu:** Bu dosyanın yapı/format kalıbı (geliştirici dokümanı + repo felsefesi formatı), [anthropics/claude-for-legal](https://github.com/anthropics/claude-for-legal) projesinden uyarlanmıştır. İçerik Türk hukukuna ve Kapital Legal'in sponsorluk pozisyonuna göre yeniden yazılmıştır. Apache 2.0 lisans yükümlülükleri için `LICENSE` ve `NOTICE` dosyalarına bakınız.

## Repo Felsefesi

1. **Anthropic'in iskeletini koru, içeriği Türkçeleştir.** Plugin yapısı (`.claude-plugin/plugin.json` + `skills/*/SKILL.md` + cold-start interview deseni) Anthropic ile birebir aynı kalır. Sadece hukuki içerik, terminoloji ve veri kaynakları Türkçeleşir.

2. **"Draft for attorney review" prensibi.** Hiçbir plugin nihai hukuki tavsiye üretmez. Her çıktı taslak halinde, kaynak gösterilerek, avukat onayına hazır şekilde sunulur. Belirsiz konularda konservatif varsayılan kullanılır.

3. **Veri kaynakları açık ve doğrulanabilir.** Her karar/madde atıfı kaynağıyla birlikte verilir. Kullanıcının resmi kaynaklardan (kvkk.gov.tr, Resmi Gazete vb.) sağladığı içerik öncelendir. Model eğitim verisinden gelen atıflar `[doğrulanmalı]` etiketi taşır. Karar metinlerinin URL'i her zaman dahil edilmelidir.

4. **Paylaşılan firma profili — kendi mekanizmamız.** Bu repo, Anthropic'in plugin başına ayrı CLAUDE.md tutmasının üstüne **kendi paylaşım mekanizmasını** kurar:

   - **Paylaşılan profil:** `~/.claude/claude-for-tr-legal/firma-profili.md` (kök seviye, plugin-bağımsız)
   - **Plugin-spesifik profil:** `~/.claude/plugins/config/claude-for-tr-legal/<plugin>/CLAUDE.md` (her plugin için ayrı)
   - **Akış:** İlk kurulan plugin'in `cold-start-interview` skill'i önce firma profilini kontrol eder. Yoksa kullanıcıdan büro bilgilerini toplar ve **paylaşılan dosyaya** yazar. Sonra plugin-spesifik soruları sorar ve **plugin-spesifik dosyaya** yazar. Sonraki plugin'lerin `cold-start-interview`'u paylaşılan dosyayı okur, kullanıcıya özet gösterip onaylatır ve şirket sorularını **tekrar sormaz** — yalnızca plugin-spesifik sorulara odaklanır.
   - **Implementation referansı:** `kvkk-uyum-tr/skills/cold-start-interview/SKILL.md` (Adım 0 ve Adım 1)
   - **Template:** `references/firma-profili-template.md` (repo kökünde)

5. **TBB Meslek Kuralları gözetilir.** Repo kök seviyede `references/tbb-meslek-kurallari-ozet.md` var. Plugin geliştirenler PR'larında TBB kontrollerini açıkça not etmelidir.

## Klasör Konvansiyonları

```
claude-for-tr-legal/                     ← repo kök
  .claude-plugin/marketplace.json        ← marketplace metadata (plugin listesi)
  LICENSE                                 ← Apache 2.0
  NOTICE                                  ← Apache 2.0 m.4 atıf yükümlülüğü
  README.md, QUICKSTART.md, KATKI.md     ← kullanıcı dokümanları
  CLAUDE.md                               ← bu dosya (geliştirici notları)

  references/                             ← REPO KÖKÜNDE paylaşılan referans dosyalar
    firma-profili-template.md
    tbb-meslek-kurallari-ozet.md
    karar-atif-kurallari.md
    baro-danisma-notu.md

  <plugin-adı>-tr/                        ← her plugin kendi klasöründe
    .claude-plugin/plugin.json            ← {name, version, description, author}
    CLAUDE.md                             ← plugin'e özel practice profile template
    README.md                             ← plugin tanıtımı + skill listesi
    skills/
      cold-start-interview/SKILL.md
      <skill-adı>/SKILL.md
    references/                           ← plugin runtime için statik veri (paketleme sırasında plugin'le birlikte yüklenir)
      firma-profili-template.md           ← (repo kökündeki ile senkron tutulur)
    hooks/                                ← (opsiyonel) lifecycle hooks
    agents/                               ← (opsiyonel) sub-agent tanımları
```

**`references/` klasörü kararı (2026-05-31 turn-6'da revize):**

Türk plugin'ler için **iki yerde** `references/` bulunur:

1. **Repo kökünde `references/`** — geliştirici dokümantasyonu ve repo-wide paylaşılan referanslar:
   - `tbb-meslek-kurallari-ozet.md` — TBB MK + Av.K. madde özetleri
   - `karar-atif-kurallari.md` — halüsinasyon disiplini
   - `baro-danisma-notu.md` — TBB reklam/tabela değerlendirmesi
   - `veri-isleme-bildirimi.md` — Anthropic API iletim çerçevesi
   - `firma-profili-template.md` — geliştirici görüntüsü için kopyası

2. **Plugin içinde `<plugin>/references/`** — plugin runtime'ında gerekli olan statik veri:
   - Şu an yalnızca `firma-profili-template.md` (cold-start template)
   - Plugin başına gerektikçe başka template/şablonlar eklenir

**Neden iki yerde?** `claude plugin install` paketleme sırasında **yalnızca plugin klasörünü** kopyalar; repo kökündeki `references/` plugin paketine dahil edilmez. Cold-start skill'inin runtime'da template'e erişebilmesi için plugin **içinde** bir kopya zorunludur. Aynı dosya geliştirici görüntüsü için kökte de bulunur.

**Senkronizasyon:** İki dosyanın aynı içeriği koruması maintainer sorumluluğundadır. Şu an `firma-profili-template.md` her iki yerde de aynı; revizyon yapılırsa iki yer de güncellenir.

**Plugin paketleme test sonucu (2026-05-31 turn-6):**
- Plugin paketi yolu: `~/.claude/plugins/cache/claude-for-tr-legal/<plugin>/<version>/`
- `${CLAUDE_PLUGIN_ROOT}` bu yola çözümlenir
- `${CLAUDE_PLUGIN_ROOT}/../` plugin'in versiyon klasörünün üstüne çıkar (yalnızca o plugin'in alt dizini, repo'nun bütünü değil)
- Dolayısıyla `${CLAUDE_PLUGIN_ROOT}/../references/` çalışmaz; **`${CLAUDE_PLUGIN_ROOT}/references/` çalışır.**

**İsimlendirme:**
- Plugin adları: küçük harf, tire ile ayrılmış, `-tr` soneki (örn. `kvkk-uyum-tr`)
- Skill adları: küçük harf, tire ile ayrılmış, Türkçe fiil tabanı tercih (örn. `aydinlatma-metni-uretici`, `karar-analizi`)
- Türkçe karakter (ı, ş, ğ, ü, ç, ö) **plugin/skill adlarında kullanılmaz** (filesystem uyumu) ama YAML/markdown içeriğinde elbette kullanılır

## SKILL.md Şablonu

```markdown
---
name: skill-adı
description: >
  Bu skill ne yapar (1-2 cümle). Hangi kullanıcı söyleminde tetiklenir.
  Örn: "ilgili kişi başvurusu", "veri sahibi talebi", "30 günlük süreyi takip et"
argument-hint: "[--bayrak açıklaması]"
---

# /<plugin>:<skill>

## Cold-start kontrolü
`~/.claude/plugins/config/claude-for-tr-legal/<plugin>/CLAUDE.md` dosyasını oku:
- Yoksa / placeholder içeriyorsa → kullanıcıyı cold-start'a yönlendir.

## İş akışı
1. Adım 1...
2. Adım 2...

## Çıktı formatı
- Avukat onayına hazır taslak
- Kaynak gösterimi: [SMK m.X], [KVKK m.Y], [Yargıtay 11.HD, X.X.2026 E.YYYY/N K.YYYY/N]
- Belirsizlik durumunda konservatif varsayılan + `[doğrulanmalı]` etiketi
- Karar atıfı verirken `references/karar-atif-kurallari.md` zorunlu kuralına uyulur

## TBB Meslek Kuralları uyumu
İlgili kural numaraları + tetikleyici kontroller
```

**Etiket disiplini (zorunlu):** Repo genelinde tek belirsizlik etiketi kullanılır: **`[doğrulanmalı]`**. İngilizce `[verify]` veya başka varyantlar kullanılmaz. Bu, kullanıcının ne yapması gerektiğini netleştirir ve tutarsızlık tetiklemez.

## Veri Kaynağı İlkesi

Bu repo, **herhangi bir üçüncü taraf MCP sunucusunu gömmez veya zorunlu kılmaz.** Skill'ler üç katmanlı çalışır:

1. **Model eğitim verisi** — Genel KVKK çerçevesi, m.10/11/12/13 yorumları, geçmiş Kurul kararları. Her atıf `[doğrulanmalı]` etiketli, kullanıcının resmi kaynaktan teyit etmesi şart.

2. **Kullanıcı girdisi** — Skill'ler kullanıcının kopyalayıp yapıştırdığı karar metnini veya verdiği URL'i `WebFetch` ile çekip analiz eder. Bu, repo'ya bağımlılık yaratmaz.

3. **Opsiyonel — kullanıcının kendi araçları** — Kullanıcı Claude.ai/Cowork'te (veya başka yerde) kendi seçtiği bir araç kurmuşsa Claude o aracı kullanabilir. Skill başında availability check yapılır; yoksa skill 1+2 modunda çalışır, hata vermez.

**Skill geliştirirken:**
- Hiçbir spesifik MCP aracı adına sabit bağımlılık yazma (örn. `mcp__falanca__filan` çağrısı yerine "kullanıcı bir karar veritabanı aracı kurduysa onu kullan" şeklinde koşullu kontrol).
- Karar atıfı eklerken **spesifik karar numarası uydurma** — `references/karar-atif-kurallari.md` sert kuralına uyulur.
- Kullanıcıyı `https://www.kvkk.gov.tr` veya ilgili resmi kaynağa yönlendir.

## Geliştirici Komutları

```bash
# Yeni plugin iskeleti oluştur (şablon kopyalama)
cp -r kvkk-uyum-tr/ <yeni-plugin>-tr/

# Skill ekle
mkdir -p <plugin>/skills/<skill-adı>
touch <plugin>/skills/<skill-adı>/SKILL.md

# Lokal test
# Claude Code'da: /plugin marketplace add <bu-repo-path>
```

## Versiyonlama

Her plugin'in `.claude-plugin/plugin.json` dosyasında semver kullanılır:
- **0.x.y** — geliştirme aşaması (P1 plugin'leri burada)
- **1.0.0** — büroda günlük kullanımda stabil
- **2.0.0** — kamu/topluluk launch

## Memory ve Çoklu Plugin Koordinasyonu

Kapital Legal'in memory sistemi (`~/.claude/projects/.../memory/`) bu repo'nun gelişiminden bağımsızdır ama bilgi paylaşır:
- `project_tr_hukuk_plugin_hub.md` — repo state, açık görevler, faz takvimi
- Her plugin tamamlandığında MEMORY.md index'ine eklenir

---
> Source: [kapital-legal/claude-for-tr-legal](https://github.com/kapital-legal/claude-for-tr-legal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
