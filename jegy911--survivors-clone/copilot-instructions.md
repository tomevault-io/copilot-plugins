## survivors-clone

> Ironfall — kod değişince docs ve yol haritasını güncelle


# Dokümantasyon disiplini (Ironfall)

Bu repoda **oyun içeriği veya mimari** değiştirildiğinde:

1. **`docs/GELISTIRICI_REHBERI.md`** — Yeni içerik *türü* (orb, silah, karakter, harita akışı, vb.) veya yeni dosya/akış varsa ilgili bölüm veya checklist ekle/güncelle.
2. **`docs/YOL_HARITASI.md`** — İlgili plan maddesi bittiyse tabloda işaretle (`[x]`) veya kaldır; **Yapılan iş günlüğü** tablosuna `YYYY-AA-GG` tarihi ve tek satır özet yaz.
3. **`docs/ERISILEBILIRLIK_VE_BAGLILIK_MATRISI.md`** — Erişilebilirlik veya bağlılık (dopamin döngüsü) hedeflerinden birini kodda karşıladıysan veya kapsam değiştiyse ilgili satırın **Durum** ve **Repo kontrolü** sütunlarını güncelle.
4. **`docs/TASARIM.md`** — Görsel, ikon, orb animasyonu, UI sunumu veya yayın maddisi tamamlandıysa veya yeni bir envanter satırı eklendiyse güncelle.
5. **`locales/en.json`** + **`locales/codex_sources/codex_extensions_en.json`** — Rutin geliştirmede yeni metin anahtarları **yalnızca İngilizce** dosyalara eklenir. `tr.json`, `zh_CN.json`, `codex_extensions_tr.json`, `codex_extensions_zh_CN.json` **donduruldu** (ikinci emre / tam dil turuna kadar güncelleme yok). `check_locale_parity.py` bu yüzden geçici olarak fark gösterebilir; dil işine girilince hizalanır.
6. **`README.md`** — Kurulum, çalıştırma veya `docs/` yapısı değiştiyse güncelle.
7. **`docs/KARAKTER_SINIFLARI_VE_TASARIM.md`** — Karakter **sınıfı** veya kahraman rol çerçevesi (co-op destek vizyonu, sınıf–ID tablosu) değiştiyse güncelle; ilgili plan satırları `YOL_HARITASI.md` içinde.
8. **`docs/lore.md`** — Evren, karakter veya düşman anlatısı netleştikçe veya yeni içerik bu lore ile bağlanacaksa ilgili bölümü güncelle.

**Yeni kahraman veya kahraman loadout işi bittiğinde:** Cevabın **sonunda** bilgi kartını şu sırayla ver (İngilizce `codex` isimleriyle uyumlu kısa açıklamalar tercih edilir): **karakter adı:** `id` — görünen ad · **karakter classı:** `hero_class` · **silah:** `start_weapon` (görünen ad — özet) · **eşya:** `start_item` veya — · **evrim silahı:** evrim ID veya —. Kahramanın kendi `characters/<id>/<id>.tscn` dosyası olmalı; başka kahraman sahnesinden kopya alındıysa `AnimatedSprite2D` **SpriteFrames** içinde animasyon **isimleri** kalsın, **frame** listeleri temizlensin (görsel sonra). Ayrıntı: `docs/GELISTIRICI_REHBERI.md` §9.

Küçük typo / tek satırlık bugfix için sadece ilgili doc’ta gerçekten yanlış bilgi varsa düzelt; her commit’te zorunlu değil.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jegy911) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
