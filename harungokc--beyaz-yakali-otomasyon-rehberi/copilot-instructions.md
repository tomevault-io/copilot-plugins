## beyaz-yakali-otomasyon-rehberi

> Bu proje, ofiste en çok zaman alan 5 işi **Claude** (Claude Code + MCP + API) ve

# Beyaz Yakalı Otomasyon Rehberi — AI Ajanlar İçin Kurulum Rehberi

Bu proje, ofiste en çok zaman alan 5 işi **Claude** (Claude Code + MCP + API) ve
**Hermes/Telegram** ile sıfırdan otomatize etmek için bir derleme/orkestrasyon
katmanıdır. n8n/Zapier zorunlu değildir — opsiyonel "ileri alternatif"tir.

> İlke: araçları biz yazmıyoruz, alanın **en iyi açık kaynak repolarını** birleştiriyoruz.
> Her otomasyonun kaynak repoları kredilenir; geri alınamaz işlemler (mail gönderme,
> takvim daveti) **insan-onaylı / taslak** varsayılanıyla kalır.

## Hızlı Kurulum

```bash
# 1. Repoyu klonla
git clone https://github.com/Harungokc/beyaz-yakali-otomasyon-rehberi.git
cd beyaz-yakali-otomasyon-rehberi

# 2. Bağımlılıkları yükle (otomasyona göre seçmeli)
pip install -r requirements.txt

# 3. .env oluştur
cp .env.example .env
# .env'i kendi anahtarlarınla doldur (en azından ANTHROPIC_API_KEY)
```

## İki Kurulum Yolu

Her otomasyon **iki şekilde** çalıştırılabilir — ikisi de aynı prompt'u kullanır:

### A) Claude ile (Claude Code + MCP)
Bilgisayar başındayken en güçlü yol. İlgili MCP server'ı bağla, `prompts/` altındaki
prompt'u kullan. Ayrıntı: [`docs/CLAUDE-KURULUM.md`](docs/CLAUDE-KURULUM.md).

```bash
# Örnek: Gmail MCP bağla (email otomasyonu için)
# Ayrıntılı liste ve komutlar → docs/CLAUDE-KURULUM.md
```

### B) Hermes / Telegram ile
Telefondan, masaüstüne bağlı kalmadan. Bir Telegram botu (kendi Hermes ajanın)
`prompts/` altındaki prompt'u sistem talimatı olarak kullanır; sen komutu Telegram'dan
yazarsın, ajan Claude API ile işler. Ayrıntı: [`docs/HERMES-KURULUM.md`](docs/HERMES-KURULUM.md).

## AI Ajan Komut Örnekleri

```
> "Gelen kutumdaki okunmamış mailleri önceliklendir, acil olanları özetle"
> "Bu toplantı kaydını özetle, kararları ve action item'ları çıkar"
> "Bu faturayı oku, tutar/tarih/satıcı bilgisini tablo olarak ver"
> "Bu haftanın satış verisinden yönetici özeti raporu çıkar"
> "Perşembe öğleden sonra 1 saatlik uygun slot bul, taslak davet hazırla"
```

## Model Notu

Kod ve prompt'lar Claude'un güncel model kimliğini kullanır: `claude-opus-4-8`.
Model id'lerini her zaman doğrula — uydurma/eski id (`claude-3-...` tarih ekli vb.) kullanma.

---
> Source: [Harungokc/beyaz-yakali-otomasyon-rehberi](https://github.com/Harungokc/beyaz-yakali-otomasyon-rehberi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
