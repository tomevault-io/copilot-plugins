## zotero-notebooklm-skill

> Zotero koleksiyonları ile NotebookLM defterleri arasında iki yönlü senkronizasyon — defter oluştur, kaynak yükle, podcast/slayt/harita üret, deep research ile genişlet


Zotero koleksiyonlarını NotebookLM defterlerine senkronize ediyorsun. Kaynakları yükle, çıktılar üret, isteğe bağlı olarak deep research ile genişlet.

## Setup

**Bu adımları ilk mesajda paralel olarak yap:**

1. **Zotero MCP kontrolü:** `ToolSearch` ile `"zotero add_by_doi"` ara. `mcp__zotero-mcp__zotero_add_by_doi` bulunamazsa: "`54yyyu/zotero-mcp` (`zotero-mcp-server`) gerekli. Kurulum: `uv tool install zotero-mcp-server`" yaz ve dur.

2. **NotebookLM MCP kontrolü:** `ToolSearch` ile `"notebooklm notebook_create"` ara. `mcp__notebooklm-mcp__notebook_create` bulunamazsa: "`jacob-bd/notebooklm-mcp-cli` gerekli. Kurulum: `uv tool install notebooklm-mcp-cli`" yaz ve dur.

3. **Auth kontrolü:** `mcp__notebooklm-mcp__server_info` çağır. Auth hatası gelirse kullanıcıyı `! nlm login` çalıştırmaya yönlendir.

Her iki MCP de hazırsa devam et.

**Mod tespiti:** Kullanıcı koleksiyon adı verdiyse hızlı mod, vermediyse interaktif mod.

---

## Adım 1 — Koleksiyon Seçimi

### Hızlı mod (koleksiyon adı verildi)
- `mcp__zotero-mcp__zotero_search_collections` ile verilen adı ara.
- Bulunamazsa `mcp__zotero-mcp__zotero_get_collections` ile tüm koleksiyonları listele, kullanıcıya göster.

### İnteraktif mod (koleksiyon adı verilmedi)
- `mcp__zotero-mcp__zotero_get_collections` ile koleksiyonları listele.
- Kullanıcıdan numara veya isimle seçim iste.

### Koleksiyon seçildikten sonra
- `mcp__zotero-mcp__zotero_get_collection_items` ile makaleleri getir.
- Eğer >15 makale varsa uyar: "Bu koleksiyonda {N} makale var. NotebookLM'de optimum 10-15 kaynak. Hepsini yükleyelim mi, yoksa alt seçim yapmak ister misin?"

---

## Adım 2 — Defter Oluşturma

- `mcp__notebooklm-mcp__notebook_list` ile mevcut defterleri kontrol et.
- Aynı isimde defter varsa sor: "'{name}' adında bir defter zaten var. Mevcut deftere ekle mi, yenisini oluştur mu?"
- Yeni defter: `mcp__notebooklm-mcp__notebook_create(title="{koleksiyon_adı}")`
- Dönen notebook_id'yi kaydet — sonraki adımlarda kullanılacak.

---

## Adım 3 — Kaynak Yükleme

Her makale için sırayla:

1. `mcp__zotero-mcp__zotero_get_item_fulltext` ile tam metni al.

2. **NotebookLM'e metin kaynağı olarak ekle:**
   `mcp__notebooklm-mcp__source_add(notebook_id="{nb_id}", source_type="text", text="{fulltext}", title="{makale_başlığı}")`

3. `mcp__zotero-mcp__zotero_get_annotations` ile anotasyonları kontrol et. Varsa:
   - Anotasyonları derle ve ayrı bir metin kaynağı olarak ekle:
   `mcp__notebooklm-mcp__source_add(notebook_id="{nb_id}", source_type="text", text="{derlenmiş_anotasyonlar}", title="Araştırmacı Notları: {makale_adı}")`

**İlerleme:** Her yükleme sonrası "{loaded}/{total} kaynak yüklendi..." yaz.

**Hata yönetimi:**
- Yüklenemeyen makaleleri atla, raporda not et.
- NotebookLM kaynak limiti (50) dolunca uyar ve `mcp__notebooklm-mcp__notebook_create` ile ikinci bir defter oluştur.

---

## Adım 4 — Readwise Entegrasyonu (opsiyonel)

**Yalnızca kullanıcı açıkça istediğinde tetiklenir** ("Readwise highlight'larımı da ekle" veya `readwise` komutu).

1. `ToolSearch` ile readwise araçlarını kontrol et. Yoksa "Readwise MCP mevcut değil, bu adım atlanıyor." de ve geç.
2. Her makale başlığı için `mcp__readwise__readwise_search_highlights` ile eşleşen highlight'ları ara.
3. Eşleşen highlight'ları makale bazında grupla, tek bir metin kaynağı olarak deftere ekle:
   `mcp__notebooklm-mcp__source_add(notebook_id="{nb_id}", source_type="text", text="{derlenmiş_highlights}", title="Readwise Highlight'ları")`
4. Eşleşme yoksa: "Readwise'da eşleşen highlight bulunamadı."

---

## Adım 5 — Çıktı Seçimi (interaktif modda)

Hızlı modda bu adımı atla.

Kullanıcıya seçenekleri sun (birden fazla seçilebilir):

| Komut | Çıktı | MCP Aracı |
|-------|-------|-----------|
| `podcast` | Audio Overview | `mcp__notebooklm-mcp__studio_create(notebook_id, artifact_type="audio")` |
| `slayt` | Slayt sunumu | `mcp__notebooklm-mcp__studio_create(notebook_id, artifact_type="slides")` |
| `video` | Video | `mcp__notebooklm-mcp__studio_create(notebook_id, artifact_type="video")` |
| `infografik` | İnfografik | `mcp__notebooklm-mcp__studio_create(notebook_id, artifact_type="infographic")` |
| `harita` | Literatür haritası | `mcp__notebooklm-mcp__notebook_query` ile metin sorgusu |
| `özet` | Karşılaştırmalı özet | `mcp__notebooklm-mcp__notebook_query` ile metin sorgusu |
| `sorgu` | Serbest soru | `mcp__notebooklm-mcp__notebook_query` |
| `yok` | Çıktı yok, defter hazır | — |

### podcast / slayt / video / infografik
- `mcp__notebooklm-mcp__studio_create(notebook_id="{nb_id}", artifact_type="{tip}", confirm=true)` ile oluştur.
- `mcp__notebooklm-mcp__studio_status(notebook_id="{nb_id}")` ile durumu kontrol et.
- Hazır olunca `mcp__notebooklm-mcp__download_artifact(notebook_id="{nb_id}", artifact_type="{tip}")` ile indir.
- Uzun sürerse: "Oluşturuluyor, 3-5 dk sürebilir. Kontrol edeyim mi?"

### harita
`mcp__notebooklm-mcp__notebook_query(notebook_id="{nb_id}", question="Bu defterdeki tüm kaynaklar arasındaki tematik bağlantıları, metodolojik yaklaşımları ve temel argümanları haritalandır. Hangi makale hangi makaleyle ne konuda örtüşüyor, nerede ayrışıyor?")` ile sor.

### özet
`mcp__notebooklm-mcp__notebook_query(notebook_id="{nb_id}", question="Bu defterdeki her kaynağın temel argümanını, metodolojisini ve bulgularını 2-3 cümleyle özetle. Sonra kaynaklar arasındaki ortak temalar ve çelişkileri belirt.")` ile sor.

### sorgu
Kullanıcının sorusunu doğrudan `mcp__notebooklm-mcp__notebook_query(notebook_id="{nb_id}", question="{kullanıcı_sorusu}")` ile gönder. Conversation ID döndüyse takip sorularında kullan.

---

## Adım 6 — Deep Research ile Genişletme (opsiyonel)

**Tetikleyici:** Kullanıcı "genişlet" veya "ilgili kaynak bul" dediğinde.

1. **Boşluk analizi:**
   `mcp__notebooklm-mcp__notebook_query(notebook_id="{nb_id}", question="Bu defterdeki kaynaklarda ele alınmayan ama konuyla doğrudan ilişkili hangi perspektifler, metodolojiler veya alt konular eksik?")`

2. **Deep research başlat:**
   `mcp__notebooklm-mcp__research_start(notebook_id="{nb_id}", query="{boşluk_analizinden_türetilen_sorgu}", mode="fast")`
   - Deep mod gerekiyorsa: `mode="deep"` (~5 dk, web araması)

3. **Durumu kontrol et:**
   `mcp__notebooklm-mcp__research_status(notebook_id="{nb_id}")`

4. **Bulunan kaynakları sun:** Numaralı liste halinde göster. Kullanıcı seçtikten sonra:
   `mcp__notebooklm-mcp__research_import(notebook_id="{nb_id}", task_id="{task_id}", indices="{seçilen}")`

5. **Zotero'ya da ekle (iki yönlü senkronizasyon):**
   - Her kaynak için `mcp__zotero-mcp__zotero_search_items` ile Zotero'da var mı kontrol et.
   - Yoksa:
     - DOI varsa: `mcp__zotero-mcp__zotero_add_by_doi(doi="{doi}")`
     - URL varsa: `mcp__zotero-mcp__zotero_add_by_url(url="{url}")`
   - Koleksiyona ata: `mcp__zotero-mcp__zotero_manage_collections(item_id="{id}", collection_id="{col_id}", action="add")`

6. **Zaman aşımı:** 5 dakikayı geçerse sor: "Deep research devam ediyor. 3 dk sonra tekrar kontrol edeyim mi?"

---

## Adım 7 — Rapor

Pipeline tamamlandığında aşağıdaki formatı göster:

```
Zotero <-> NotebookLM Raporu
Koleksiyon: {koleksiyon_adı}
Defter: {defter_adı} (ID: {nb_id})
Yüklenen kaynaklar: {N}/{total}
  - Metin olarak: {n1}
  - Anotasyon kaynağı: {n2}
Readwise: {eklendi / kullanılmadı}
Oluşturulan çıktılar: {liste}
Deep Research - yeni kaynaklar: {N} {liste}
Atlanan/hata: {liste veya "Yok"}
```

---

## İnteraktif Komutlar

Pipeline sırasında veya sonrasında kullanıcı bu komutları verebilir:

| Komut | İşlev |
|-------|-------|
| `listele` | Zotero koleksiyonlarını göster |
| `seç N` | N numaralı koleksiyonu seç |
| `hepsini yükle` | Tüm makaleleri onay sormadan yükle |
| `podcast` | Audio Overview oluştur |
| `slayt` | Slayt sunumu oluştur |
| `video` | Video oluştur |
| `infografik` | İnfografik oluştur |
| `harita` | Literatür haritası |
| `özet` | Karşılaştırmalı özet |
| `sorgu "..."` | Deftere soru sor |
| `genişlet` | Deep research ile yeni kaynak bul |
| `readwise` | Readwise highlight'larını dahil et |
| `durum` | Çıktıların oluşturma durumunu kontrol et (`studio_status`) |
| `indir` | Hazır çıktıları indir (`download_artifact`) |
| `dur` | Mevcut adımı durdur |
| `atla` | Mevcut adımı geç |
| `rapor` | Şu ana kadar yapılanların özetini ver |

---
> Source: [orhoncan/zotero-notebooklm-skill](https://github.com/orhoncan/zotero-notebooklm-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
