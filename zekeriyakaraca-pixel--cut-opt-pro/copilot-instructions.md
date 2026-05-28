## cut-opt-pro

> Bu dosya, CutOpt Pro v2.x projesinin genel mimari yapısını, klasör hiyerarşisini, kod bağımlılıklarını ve geliştirme prensiplerini açıklamaktadır.

# CutOpt Pro - Proje Mimarisi ve Geliştirici Rehberi

Bu dosya, CutOpt Pro v2.x projesinin genel mimari yapısını, klasör hiyerarşisini, kod bağımlılıklarını ve geliştirme prensiplerini açıklamaktadır.
Yeni özellik geliştirirken veya hata ayıklarken bu rehberdeki mimari kurallara uyulması esastır.

---

## Genel Bakış

**CutOpt Pro**, profil ve boru kesim işlemleri için geliştirilmiş **1-Boyutlu Kesim Stok Problemi (1D Cutting Stock Problem)** optimizasyon yazılımıdır.
Proje, eski monolitik yapısından (v1) çıkarak **Modüler Mimari (v2)** yapısına geçiş yapmıştır.

- **Frontend** (PySide6 / Qt) ve **Backend** (İş Mantığı, Algoritmalar, Raporlama) birbirinden tamamen izole edilmiştir.
- Sistem çok dilli destek (i18n / TR + EN), CNC entegrasyonu (DSTV/NC1), lisanslama ve MCP tabanlı AI entegrasyonu içermektedir.
- Ticari bir üründür: Trial / Basic / Professional lisans katmanları mevcuttur.

---

## Klasör Hiyerarşisi

```text
kesim_qt_build/
│
├── main.py                         # Uygulama giriş noktası; QApplication, MainApp(QMainWindow),
│                                   # QUndoStack, sekme yönetimi, lisans kontrolü, autosave timer
├── app_config.json                 # Uygulama ayarları: tema, dil, aktif proje yolu, otomatik kayıt aralığı
├── profil_database.json            # Profil geometri veritabanı (IPE, HEA, SHS vb. h/b/tw/tf/kg_m)
├── requirements.txt                # Python paket bağımlılıkları (sürüm kısıtlamalı)
├── CutOptPro.spec                  # PyInstaller konfigürasyonu (hidden imports, data files, PySide6)
├── RELEASE_BUILD.bat               # Yayın sürümü derleme betiği
├── setup_script.iss                # Inno Setup Windows yükleyici scripti
├── LICENSE.txt                     # Ticari yazılım lisans metni
├── E-378.nc1                       # DSTV/NC1 format örnek dosyası (test için)
├── COP-*.lic                       # Test lisans dosyaları (dev, trial, basic)
│
├── (Yardımcı Scriptler - Kök)
│   ├── add_international_profiles.py  # Profil veritabanına TR/EN çeviri ekler
│   ├── enrich_db.py                   # Profil veritabanını kg/m değerleriyle zenginleştirir
│   ├── gen_ses.py                     # Oturum üretici yardımcısı
│   ├── fix_script.py                  # Veri onarım aracı
│   └── create_icon.py                 # Uygulama ikonu üretici
│
├── backend/                        # [İş Mantığı, Veri, Raporlar, Algoritmalar — Qt BAĞIMSIZ]
│   │
│   ├── core/                       # v2 Modüler Mimari Çekirdeği
│   │   │
│   │   ├── algorithms/             # Optimizasyon Motorları
│   │   │   ├── optimizer.py        # Ana koordinatör; problem boyutuna göre algoritma otomatik seçer
│   │   │   ├── greedy.py           # FFD / BFD / WFD + 2-opt post-processing
│   │   │   ├── column_generation.py# Column Generation + Linear Programming (scipy)
│   │   │   ├── genetic.py          # Genetik Algoritma (OX-crossover, swap+reverse mutasyon)
│   │   │   ├── branch_bound.py     # Branch & Bound (küçük örnekler için)
│   │   │   └── common.py           # Algoritmalar arası paylaşılan yardımcılar
│   │   │
│   │   ├── business/               # Servis Katmanı (Qt-free, Unit Test edilebilir)
│   │   │   ├── optimization_service.py  # Optimizasyon iş akışı orkestrasyonu
│   │   │   ├── stock_service.py         # Stok/envanter yönetimi
│   │   │   ├── project_service.py       # Çoklu proje CRUD işlemleri
│   │   │   ├── stats_service.py         # Dashboard istatistikleri (verimlilik, fire, maliyet)
│   │   │   ├── cost_service.py          # Malzeme maliyet hesaplamaları
│   │   │   ├── batch_service.py         # Toplu (batch) operasyonlar
│   │   │   └── command_service.py       # Undo/redo komutları için iş mantığı
│   │   │
│   │   └── data/                   # Veri Katmanı
│   │       ├── models.py           # Dataclass'lar: StokItem, ParcaItem, ProjeData, KesimSonucu
│   │       ├── repository.py       # Repository Pattern; JSON + Excel dosya I/O, atomik yazma
│   │       └── validators.py       # Giriş doğrulama kuralları
│   │
│   ├── i18n/                       # Çok Dilli Destek
│   │   ├── __init__.py             # i18n motoru; tr/en JSON tabanlı, Türkçe fallback
│   │   ├── tr.json                 # Türkçe çeviriler (~19KB)
│   │   └── en.json                 # İngilizce çeviriler (~17KB)
│   │
│   ├── reports/                    # Raporlama Modülleri
│   │   ├── base.py                 # PDF/Excel raporlayıcıları için abstract taban sınıf
│   │   ├── excel/
│   │   │   ├── exporter.py         # Koşullu formatlı, grafikli Excel export
│   │   │   ├── parser.py           # Excel veri okuma
│   │   │   └── styles.py           # Hücre stilleri ve formatlama
│   │   ├── pdf/
│   │   │   ├── cutting_card.py     # Operatör kesim kartı (ReportLab)
│   │   │   ├── material_list.py    # Malzeme listesi PDF
│   │   │   └── visual_plan.py      # Görsel kesim planı PDF
│   │   └── txt/
│   │       ├── cutting_report.py   # Metin tabanlı kesim raporu
│   │       ├── material_report.py  # Metin tabanlı malzeme raporu
│   │       └── operator_card.py    # Metin tabanlı operatör kartı
│   │
│   ├── utils/                      # Yardımcı Araçlar
│   │   ├── license_manager.py      # RSA imzalı lisans doğrulama, makine kilidi, katman kısıtlamaları
│   │   ├── autosave_manager.py     # Kurtarma dosyası yönetimi (.kesim_recovery.json)
│   │   ├── backup_manager.py       # Meta verili versiyonlu yedek yönetimi
│   │   ├── error_handler.py        # Global exception handler + wrap_slot dekoratörü
│   │   ├── theme_manager.py        # Koyu/açık tema değiştirme
│   │   ├── file_operations.py      # Güvenli dosya yolu operasyonları
│   │   ├── qr_generator.py         # Proje QR kodu üretimi
│   │   └── profile_cache.py        # Profil lookup önbellekleme
│   │
│   ├── nc1_parser.py               # DSTV/NC1 dosya ayrıştırıcı (profil spec, delikler, boyutlar)
│   ├── cnc_export_manager.py       # Optimizasyon sonuçlarını NC1 formatına dönüştürür
│   ├── drive_sync.py               # Google Drive bulut senkronizasyonu
│   │
│   └── (Legacy — v1 Dosyaları, REFERANS ALINMAMALI)
│       ├── algoritma.py            # v1 monolitik optimizasyon (~59KB, geriye uyumlu wrapper)
│       ├── veri.py                 # v1 veri persistansı (MCP bridge hâlâ kullanmaktadır)
│       ├── config.py               # Yollar, renk temaları, profil standartları, kg/m tablosu
│       ├── excel_io.py             # Excel import/export
│       ├── pdf_export.py           # PDF üretimi (eski)
│       ├── dogrula.py              # Veri doğrulama (kalite sınıfları, profil tipleri)
│       ├── template_manager.py     # Rapor şablon yönetimi
│       ├── export_manager.py       # Üst düzey export koordinasyonu
│       └── log.py                  # Loglama konfigürasyonu
│
├── frontend/                       # [UI Katmanı — PySide6 BAĞIMLI]
│   │
│   ├── state/
│   │   ├── app_state.py            # Qt-free AppState dataclass (merkezi uygulama durumu)
│   │   └── signals.py              # AppSignals Singleton — Loose Coupling sinyal merkezi
│   │
│   ├── ui_stok.py                  # Stok/envanter yönetim ekranı (tablo, filtre, arama, maliyet)
│   ├── ui_parca.py                 # Parça yönetim ekranı
│   ├── ui_optimizasyon.py          # Optimizasyon kontrolü, algoritma seçimi, görsel kesim planı (QPainter)
│   ├── ui_dashboard.py             # Proje istatistikleri, verimlilik metrikleri, maliyet özeti
│   ├── ui_delik_plani.py           # NC1 delik/delgi planı görselleştirme ve yönetimi
│   │
│   ├── commands.py                 # QUndoCommand alt sınıfları:
│   │                               # StokEkleCommand, StokSilCommand, StokGuncelleCommand,
│   │                               # ParcaEkleCommand, ParcaSilCommand, ParcaGuncelleCommand,
│   │                               # ProjeGuncelleCommand, OptimizasyonKaydetCommand
│   │
│   ├── widgets/                    # Özelleştirilmiş Widget Kütüphanesi
│   │   ├── __init__.py             # Kolaylık importları
│   │   ├── buttons.py              # Özel buton stilleri (qt_btn, qt_icon_btn, qt_action_btn)
│   │   ├── tables.py               # Tablo widget'ları, delegate'ler, modeller
│   │   ├── notifications.py        # Toast bildirimleri (success, warning, error, info)
│   │   ├── search_bar.py           # Filtreli arama widget'ı
│   │   ├── selectors.py            # Profil/kalite/standart combo box'ları
│   │   ├── progress_dialog.py      # İptal destekli modal ilerleme diyaloğu
│   │   └── styles.py               # Tema CSS factory'leri (TABLE_STYLE, APP_STYLE)
│   │
│   ├── dialogs/                    # Pop-up Diyaloglar
│   │   ├── license_dialog.py       # Lisans aktivasyon/durum diyaloğu
│   │   ├── license_guard.py        # check_feature() / check_limit() — katman kısıtlama uygulayıcı
│   │   ├── cnc_settings_dialog.py  # NC1 export konfigürasyon diyaloğu
│   │   └── missing_profile_dialog.py # Tanımsız profil işleyici
│   │
│   └── widgets_legacy.py           # v1'den taşınmış, refactor bekleyen eski bileşenler
│
├── controllers/                    # UI ve Backend Arasındaki Köprüler
│   ├── stok_controller.py          # Stok CRUD, filtreleme, sıralama, Excel senkronizasyonu
│   ├── parca_controller.py         # Parça CRUD ve doğrulama
│   ├── optimizasyon_controller.py  # Algoritma seçimi, QThread'de optimizasyon çalıştırma
│   └── delik_controller.py         # NC1 import/export koordinasyonu
│
├── agent/                          # AI Entegrasyonu — Model Context Protocol
│   ├── server.py                   # FastMCP sunucusu; araç yeteneklerini dışarıya açar
│   └── bridge.py                   # AgentBridge; MCP komutlarını backend servislerine map'ler (Qt-free)
│
├── tools/                          # Araçlar ve Lisans Yönetimi
│   └── keygen.py (vb.)             # Lisans üretici, doğrulayıcı, test araçları
│
├── keys/                           # RSA şifreleme/imzalama anahtarları (lisanslama)
│
├── tests/                          # Pytest Tabanlı Testler (25+ dosya)
│   ├── test_models.py              # Dataclass doğrulama
│   ├── test_repository.py          # Dosya I/O, JSON/Excel senkronizasyonu
│   ├── test_validators.py          # Giriş doğrulama kuralları
│   ├── test_optimization_service.py
│   ├── test_stats_service.py
│   ├── test_cost_service.py
│   ├── test_batch_service.py
│   ├── test_command_service.py
│   ├── test_autosave_manager.py
│   ├── test_backup_manager.py
│   ├── test_error_handler.py
│   ├── test_file_operations.py
│   ├── test_qr_generator.py
│   ├── test_theme_manager.py
│   ├── test_profile_cache.py
│   ├── test_stok_controller.py
│   ├── test_parca_controller.py
│   ├── test_optimizasyon_controller.py
│   ├── test_delik_controller.py
│   ├── test_ui_state.py
│   ├── test_ui_widgets.py
│   ├── test_search_bar.py
│   ├── test_reports.py
│   ├── test_nc1_parser.py
│   ├── test_integration_proje.py   # Uçtan uca entegrasyon: proje oluşturma → optimizasyon → export
│   ├── test_phase_c.py
│   ├── comprehensive_test.py       # Kapsamlı entegrasyon test koşucusu
│   └── test_suite.py               # Yardımcı test paketi
│
├── docs/                           # Dokümantasyon
│   ├── V2_MIMARI_PLAN.md           # v2 mimari detay planı
│   ├── V2_GUNCELLEME_PLANI.md      # v2 geçiş/güncelleme planı
│   ├── REFACTORING_PLAN.md         # Kod modernizasyon stratejisi
│   ├── PHASE_C_COMPLETION_SUMMARY.md # Özellik tamamlanma durumu
│   ├── GUNCELLEME_TAKIP.md         # Güncelleme takip logu
│   ├── LISANS_MEKANIZMASI_PLANI.md # Lisanslama sistemi tasarımı (RSA imzalama, katman kısıtlamaları)
│   ├── LISANS_URETIM.md            # Lisans üretim aracı dokümantasyonu
│   ├── LISANS_KULLANIM.md          # Son kullanıcı lisans kullanım rehberi
│   ├── NC1_DELIK_PLANI_UYGULAMA_PLANI.md  # CNC entegrasyon yol haritası
│   ├── cnc_entegrasyon_plani.md    # CNC iş akışı uygulama planı
│   ├── guvenlik_raporu.md          # Güvenlik denetim raporu
│   ├── DSTV_Profiles/              # Profil teknik şartname belgeleri
│   └── LC53-IPE200/                # Örnek proje yapısı (gerçek kesim verisi içerir)
│
├── MCP_UPLOAD/                     # agent/ klasörünün dağıtım/yükleme kopyası
├── RELEASE/                        # Yayın derlemeleri ve artefaktları
├── dist/CutOptPro/                 # PyInstaller çıktısı (derlenmiş exe yapısı)
├── .claude/                        # Claude Code konfigürasyonu (planlar, ayarlar)
└── .vscode/                        # IDE konfigürasyonu
```

---

## Mimari Prensipler ve Kod Bağımlılıkları

### 1. Separation of Concerns (Sorumlulukların Ayrılması)

- **Backend, Frontend'i bilmez:** `backend/` altındaki hiçbir dosyada `PySide6` (veya Qt) importu yapılamaz. Bu kural Unit Test edilebilirliği maksimize eder.
- **Veri ve UI ayrımı:** UI modülleri (`ui_*.py`) doğrudan dosyaya/veritabanına yazmaz. Tüm veri işlemleri Controller'lar ve `backend/core/business` servisleri üzerinden yapılır.

### 2. Merkezi Durum ve Olay Yönetimi

**`AppState`** (`frontend/state/app_state.py`) — Qt-free dataclass:

```python
@dataclass
class AppState:
    proje_dir: str          # Aktif proje dizin yolu
    proje_adi: str          # Proje adı
    data: Dict              # Ham proje verisi {stok, parcalar, proje}
    son_sonuclar: Dict      # Son optimizasyon sonuçları
    hafiza: Dict            # Oturum belleği (geçici veri)
    aktif_sayfa: str        # Aktif sekme ("stok" | "parca" | "optimizasyon" | ...)
    tema: str               # "dark" | "light"
    is_dirty: bool          # Kaydedilmemiş değişiklik var mı
    otomatik_kayit_aralik: int  # Otomatik kayıt aralığı (saniye)
```

**`AppSignals`** Singleton (`frontend/state/signals.py`) — Loose Coupling sinyal merkezi:

```text
Veri Değişikliği:  stok_degisti | parca_degisti | proje_degisti
Proje Yaşam Döngüsü: proje_yuklendi(str) | proje_kaydedildi | proje_kapatildi
Optimizasyon:      optimizasyon_basladi | optimizasyon_ilerlemesi(str, int) |
                   optimizasyon_tamamlandi(dict) | optimizasyon_hatasi(str) | optimizasyon_iptal
UI Navigasyon:     sayfa_degisti(str) | bildirim(str, str) | tema_degisti(str) | dil_degisti(str)
Undo/Redo:         undo_stack_degisti(bool, bool)
Autosave:          dirty_degisti(bool) | otomatik_kaydedildi(str) | kurtarma_mevcut(str)
Toplu İşlem:       toplu_islem_ilerleme(int, int, str) | toplu_islem_tamamlandi(int, list)
CNC/Delik:         delikler_degisti | delik_plani_yenile(str)
```

### 3. Service & Repository Pattern

- **Repository:** Veri erişimi `DataRepository` üzerinden yapılır. JSON + Excel çift kaynak desteği, atomik yazma (geçici dosya → yeniden adlandır) sağlar.
- **Business Services:** Her iş alanı için ayrı servis: `OptimizationService`, `StatsService`, `ProjectService`, `CostService`, `BatchService`. Controller'lar bu servisleri çağırır.

### 4. Undo / Redo Sistemi

- `main.py` içinde global bir `QUndoStack` bulunur.
- Her veri değiştirici işlem (`frontend/commands.py` altındaki `QUndoCommand` alt sınıfı) stack'e itilir.
- Komutlar çalışırken hem veriyi günceller hem de ilgili sinyali fırlatır.

### 5. Optimizasyon Algoritması Otomatik Seçimi

`backend/core/algorithms/optimizer.py` problem boyutuna göre algoritma seçer:

| Benzersiz Profil Sayısı | Çalışan Algoritmalar |
|------------------------|---------------------|
| n ≤ 15 | B&B + Column Generation + Genetic + Greedy |
| 15 < n ≤ 60 | Column Generation + Genetic + Greedy |
| n > 60 | Genetic + Greedy |

Her algoritma aynı arayüzü döndürür: `[{"cubuk_no", "kesimler", "fire", "verimlilik"}, ...]`

### 6. Uluslararasılaştırma (i18n)

- Türkçe ve İngilizce desteklenir; dinamik dil değiştirme mevcuttur.
- Metinler asla hardcoded yazılmaz; `backend/i18n/tr.json` ve `en.json` üzerinden yüklenir.

### 7. CNC & NC1 Entegrasyonu

- `nc1_parser.py`: DSTV dosyalarını okur → profil spec, delikler, boyutlar çıkarır.
- `cnc_export_manager.py`: Optimizasyon sonuçlarını NC1 formatına dönüştürür.
- `delik_controller.py`: Bu sürecin UI bağlantısını sağlar; `ui_delik_plani.py` ile koordineli çalışır.

### 8. Lisanslama Sistemi

`backend/utils/license_manager.py` — Makine kilitli, RSA imzalı lisanslar:

| Özellik | Trial | Basic | Professional |
|---------|-------|-------|--------------|
| Excel Export | — | ✓ | ✓ |
| PDF Export | — | ✓ | ✓ |
| Toplu (Batch) Import | — | — | ✓ |
| Google Drive Sync | — | — | ✓ |
| Maks. Parça Sayısı | 100 | Sınırsız | Sınırsız |
| Maks. Proje Sayısı | 5 | Sınırsız | Sınırsız |
| Süre | 30 gün | Süresiz | Süresiz |

Makine ID'si: anakart seri no (birincil) + disk seri no (yedek) kombinasyonundan türetilir.
`frontend/dialogs/license_guard.py` → `check_feature()` / `check_limit()` ile UI katmanında kısıtlamalar uygulanır.

---

## Kod Akışı (Code Flow) Örnekleri

### Stok Ekleme Süreci
1. Kullanıcı `ui_stok.py` ekranından bilgileri girer ve "Ekle"ye basar.
2. UI, `stok_controller.py` içindeki `stok_ekle(kayit, undo_stack)` metodunu çağırır.
3. Controller, `StokEkleCommand` (`frontend/commands.py`) nesnesi oluşturur ve `undo_stack`'e iter.
4. Komut çalıştığında (`redo`):
   - Veri, `DataRepository` aracılığıyla `AppState`'e/backend'e yazılır.
   - `signals.stok_degisti.emit()` fırlatılır.
   - `AppState.mark_dirty()` çağrılarak autosave tetiklenir.
5. `ui_stok.py` sinyali yakalayıp tabloyu günceller.

### Optimizasyon Çalıştırma Süreci
1. Kullanıcı `ui_optimizasyon.py` ekranından algoritmayı seçer ve başlatır.
2. UI kilitlenmemesi için `QThread` başlatılır; `signals.optimizasyon_basladi.emit()` fırlatılır.
3. `OptimizasyonController`, `backend/core/algorithms/optimizer.py`'yi çağırır; problem boyutuna göre algoritma otomatik seçilir.
4. Sonuçlar `OptimizationService` üzerinden belleğe kaydedilir.
5. `signals.optimizasyon_tamamlandi.emit(sonuclar)` fırlatılır; UI güncellenip PDF/Excel export seçenekleri aktif hale gelir.

### Lisans Kontrolü
1. Uygulama başlangıcında `main.py`, `LicenseManager.dogrula()` çağırır.
2. `LicenseResult(durum, plan, ozellikler, kalan_gun, mesaj)` döner.
3. Kısıtlı özellikler için `license_guard.check_feature(widget, app, "excel_export")` çağrısı yapılır.
4. Trial süresi dolmuşsa veya özellik mevcut planda yoksa upgrade diyaloğu gösterilir.

### MCP AI Akışı
1. Dış AI aracı (Claude Desktop vb.) `agent/server.py` üzerindeki MCP araçlarını çağırır.
2. `AgentBridge` (`agent/bridge.py`) bu çağrıları backend servislerine map'ler (Qt bağımlılığı yoktur).
3. Sonuçlar JSON olarak AI aracına döner.

---

## MCP Araçları (agent/server.py)

| Araç | Açıklama |
|------|----------|
| `list_projects()` | Tüm projeleri meta veriyle listeler |
| `select_project(name)` | Aktif projeyi değiştirir |
| `import_excel_parts(file_path)` | Excel'den toplu parça aktarımı |
| `run_optimization(profile_name?)` | Optimizasyonu çalıştırır |
| `get_cut_report(profile_name)` | Profil için metin raporu döner |
| `export_results_to_excel(file_path)` | Tam raporu Excel'e aktarır |

---

## Temel Bağımlılıklar (requirements.txt)

| Paket | Sürüm | Kullanım Amacı |
|-------|-------|----------------|
| `PySide6` | ≥ 6.10.0 | Qt6 GUI çerçevesi |
| `numpy` | ≥ 2.0.0 | Sayısal hesaplama |
| `pandas` | ≥ 3.0.0 | Veri manipülasyonu |
| `matplotlib` | ≥ 3.10.0 | Grafik / görselleştirme |
| `scipy` | ≥ 1.11.0 | Linear Programming (Column Generation) |
| `reportlab` | ≥ 4.0.0 | PDF üretimi |
| `openpyxl` | ≥ 3.1.0 | Excel okuma/yazma |
| `qrcode[pil]` | ≥ 7.4.0 | QR kod üretimi |
| `Pillow` | ≥ 10.0.0 | Görüntü işleme |
| `cryptography` | ≥ 41.0.0 | RSA imzalama (lisanslama) |
| `google-api-python-client` | ≥ 2.100.0 | Google Drive entegrasyonu |
| `google-auth-httplib2` | ≥ 0.1.0 | Google OAuth yardımcısı |
| `google-auth-oauthlib` | ≥ 1.1.0 | Google OAuth akışı |
| `mcp` | — | FastMCP AI aracı sunucusu |
| `pytest` | — | Birim ve entegrasyon testleri |
| `pyinstaller` | — | Windows .exe paketleme |

---

## Veri Persistans Stratejisi

| Format | Dosya | Kullanım |
|--------|-------|----------|
| JSON | `stok_data.json` | Stok, parçalar, proje meta verisi (birincil) |
| Excel | `stok.xlsx` | Tamamlayıcı; çakışmada öncelikli |
| NC1 | `*.nc1` | CNC makineleri için DSTV formatı |
| Yedek | `yedekler/` | Versiyonlu yedekler + meta veri |
| Kurtarma | `.kesim_recovery.json` | Çökme kurtarma dosyası |
| Config | `app_config.json` | Kullanıcı tercihleri ve uygulama ayarları |

Her proje `%APPDATA%\CutOptPro\projeler\<proje_adi>\` altında ayrı bir dizinde saklanır.

---

## Geliştirici Yönergeleri

### Yeni Özellik Eklerken İzlenecek Adımlar

1. **İş Mantığı:** Çekirdek hesaplamayı `backend/core/business/` veya `backend/utils/` altında Qt-free bir servis olarak yaz.
2. **Test:** `tests/test_<ozellik>.py` dosyası oluştur. Hedef: **%80+ test coverage**.
3. **State & Sinyal:** Uygulama durumunu değiştiriyorsa `AppState`'e alan ekle; sinyal gerekiyorsa `signals.py`'a tanımla.
4. **Arayüz:** İlgili `ui_*.py` dosyasını güncelle veya `frontend/widgets/` altında yeni bir bileşen oluştur.
5. **Lisans Koruması:** Ticari özellikse `license_guard.check_feature()` ile koru.
6. **Hata Yönetimi:** `backend/utils/error_handler.py` ve `notifications.py` toast sistemi kullan.
7. **Çeviri:** Yeni metin varsa `backend/i18n/tr.json` ve `en.json` dosyalarına ekle.

### Klasör Kuralları

- Tek bir dosyayı çok işlevli yapmaktan kaçın.
- Birden fazla sayfada kullanılan bileşeni `frontend/widgets/` altına taşı.
- **Legacy Dosyalar:** `backend/` altındaki `algoritma.py`, `excel_io.py`, `pdf_export.py` gibi v1 dosyaları ve `frontend/widgets_legacy.py` refactor tamamlanana kadar yeni geliştirmelerde **referans alınmamalıdır**. Tek istisna: `backend/veri.py` — MCP bridge hâlâ bunu kullanmaktadır.

### Hata Ayıklama ve Loglar

- Global hatalar `%APPDATA%\CutOptPro\logs\app.log` dosyasına yazılır.
- Yeni slot'lar için `@wrap_slot` dekoratörünü (`backend/utils/error_handler.py`) kullan.
- Tüm hata fırlatmalarında `backend/utils/error_handler.py` sisteminin devreye girdiğinden emin ol.

### Test Çalıştırma
```bash
pytest tests/                    # Tüm testler
pytest tests/ --cov=backend      # Coverage raporu ile
pytest tests/test_models.py -v   # Tek dosya
```

---

## MCP (Model Context Protocol) AI Entegrasyonu

Proje `agent/` altında dış AI araçlarıyla iletişim kurmak için bir MCP Sunucusu barındırır:

- **`agent/server.py`:** FastMCP konfigürasyonu. Araç yeteneklerini dışarıya (Claude Desktop vb.) açar.
- **`agent/bridge.py`:** `AgentBridge` sınıfı — MCP komutlarını `backend/veri.py` ve core business servislerine map'ler. Qt bağımlılığı yoktur.

> Not: `agent/bridge.py` hâlâ `backend/veri.py` (legacy) kullanmaktadır. İleride `ProjectService` + `OptimizationService` üzerine geçirilmesi planlanmaktadır.

---

**Mimar: CutOpt Pro Geliştirme Ekibi & AI Assistant**
**Versiyon Uyumluluğu: V2.4+ Modüler Mimari**
**Son Güncelleme: 2026-05-07**

---
> Source: [zekeriyakaraca-pixel/Cut_opt_pro](https://github.com/zekeriyakaraca-pixel/Cut_opt_pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
