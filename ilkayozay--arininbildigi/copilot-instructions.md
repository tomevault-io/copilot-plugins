## arininbildigi

> Bu depoda çalışan tüm ajanlar (Copilot, Codex, Claude Code vb.) bu kurallara uyar.

# AGENTS.md — Çalışma Protokolü

Bu depoda çalışan tüm ajanlar (Copilot, Codex, Claude Code vb.) bu kurallara uyar.
Bu dosya **protokol**dür; ton/üslup/yöntem kararlarını TEKRARLAMAZ.

## En üst referans
1. `00-kitap-anayasasi.md` — ton, üslup, yöntem, sesler, tez. **Her yazım kararının üstünde.**
2. `01-yol-haritasi.md` — bölüm bazında kanonik plan ve durum.
Çelişkide anayasa kazanır. Bir bölüme başlamadan önce ikisi de okunur.

## Proje
Türkçe popüler-bilim kitabı. Çalışma adı: **Arının Bildiği**. Yapay zekânın omurgasını
(vektör→embedding→…→transformer→LLM→ötesi) tek bir tez etrafında, sıfırdan kurar:
*bir zekânın ödediği bedel nereye saklanır?*

## Dizin düzeni
- `chapters/` — kaynak kitap metni; **gerçek yazım burada** (Markdown).
- `assets/` — `style.css` ve `fonts/`; PDF/HTML görünümü buradan beslenir. Stil ve görsel girdiler burada tutulur.
- `build/` — üretilmiş çıktılar (PDF/EPUB/HTML). **Elle düzenlenmez**, derleyici üretir.
- `notes/` — çalışma notları. `references/` — kaynaklar.
- Kök: `00-kitap-anayasasi.md`, `01-yol-haritasi.md`, `build.py`.

## Bölüm dosyaları (chapters/)
`00-onsoz` · `10-dunya-nasil-sayiya-doner` · `20-anlamin-koordinatlari-embedding` ·
`30-sikistirma-ve-latent-uzayin-dogusu` · `40-noktadan-dagilima-hayal-gucunun-dogusu` ·
`50-dizi-ve-hafizanin-acisi` · `60-attention-onemli-olana-odeme-yapmak` ·
`70-transformer-iliski-makinesi` · `80-llm-kelimelerden-kurulu-dunya-modeli` ·
`90-cok-duyulu-makine-multimodal` · `100-erisim-hafiza-depo-olarak-dunya` ·
`110-ajanlar-araclar-beden` · `120-maliyetin-fizigi-verimlilik-cephesi` ·
`130-catlaklarin-haritasi` · `140-arka-kapak`
Dosya adındaki sayı öneki yalnız sıralama içindir; kanonik bölüm sırası ve adı 01-yol-haritasi.md içinde tanımlanır.

## Yazım disiplini (özet — ayrıntı anayasada)
- **Berraklık birinci kuraldır:** hiçbir cümle iki kez okutmaz; devrik/kayan özne yok.
- Her kavram **acı sırasıyla**: acı → sezgi → matematiğin adı → kod(KUTU) → çatlak(ÇATLAK) → defter(DEFTER).
- Her fikir bir **sahneye** bağlanır; ders-notu tonu yasak. Sıkışan bölüm ikiye bölünür.
- Bloklar: KUTU, DEFTER, VAKA, ÇATLAK, DERİN, ŞEKİL — anayasadaki tanımlarıyla.
- Uydurma alıntı yok; sayılarda mertebe namusu; tarihsel aşırı-kesinlik yok.

## Değişiklik kuralları
- Seçili/işaret edilen bölüm istenmişse **yalnız o dosya** değişir.
- Mevcut üslup korunur; metin gereksiz yere baştan icat edilmez.
- Dosyayı değiştirmeden önce **neyin değişeceği bir cümleyle** söylenir.
- Edit talebi formatı: *"chapters/<dosya>: '<aynen alıntı>' → '<yeni>'"*. Nokta atışı uygulanır.

## Derleme
Markdown kaynaklardan tek PDF üretilir:
```
pip install weasyprint markdown pymdown-extensions pygments
python build.py        # build/arinin-bildigi.pdf
```
`build.py` içindeki MANIFEST bölüm sırasını ve TOC açıklamalarını tutar; yeni bölüm
hem `chapters/`e dosya hem MANIFEST'e bir satır olarak eklenir. `:::` blok sözdizimi:
`::: kutu | Başlık` … `:::` (kutu/vaka/catlak/derin/defter).

---
> Source: [ilkayozay/arininbildigi](https://github.com/ilkayozay/arininbildigi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
