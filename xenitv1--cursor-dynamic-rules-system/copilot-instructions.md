## rules

> Merkezi kural yönetim sistemi - Dinamik kural seçimi ve component'ler arası koordinasyon


# Cursor AI Assistant - Merkezi Kural Yönetimi

## NASIL ÇALIŞIR? - SİSTEM AKIŞ DİYAGRAMI

```
Kullanıcı İsteği
       ↓
1. CONTEXT DETECTION (Otomatik Analiz) ⚠️ ZORUNLU
       ↓
2. Windows 11 & PowerShell Kontrolü 🖥️
       ↓
3. GitHub İzin Kontrolü 🔒
       ↓
4. KURAL SETİ SEÇİMİ (Dinamik)
       ↓
5. İLGİLİ COMPONENT'LERİ YÜKLEDİL (Modüler)
       ↓
6. UYGULAMA (Fokuslu)
```

## TEMEL PRENSİPLER - HER ZAMAN AKTİF

### 🇹🇷 DİL KURALI (MUTLAK)
- **Kullanıcıyla iletişim**: Her zaman Türkçe
- **Kod ve teknik**: İngilizce
- **Dosya yapısı**: İngilizce

### 📏 DOSYA BOYUTU KURALI (MUTLAK - İSTİSNASIZ)
- **HARD LİMİT**: 700-1000 satır
- **BLOKAJ**: 700+ satırda YENİ KOD YOK
- **ZORUNLU**: Refactoring gerekli
- **ACİL DURUM**: Sadece kritik hatalar için +50 satır

### 🧹 TEMİZ KOD (MUTLAK)
- **Kullanılmayan importları sil**
- **Gereksiz kodu temizle**
- **Tek sorumluluk ilkesi**
- **Minimal implementasyon**

### 🖥️ SİSTEM VE GÜVENLİK KURALLARI (MUTLAK - UNUTULMAZ)
- **İşletim Sistemi**: Windows 11 kullanıcısı - PowerShell komutları Windows'a uygun olmalı
- **GitHub Güvenliği**: Kullanıcı izni olmadan ASLA GitHub'a yükleme yapılmaz
- **Dinamik Kural Seçimi**: Her talepte önce CONTEXT DETECTION yapılır, uygun kural seti seçilir
- **Bu kurallar sistemin başında uygulanır, sonradan kontrol edilmez**

## CONTEXT DETECTION - OTOMATİK ANALİZ SİSTEMİ (ZORUNLU ADIM)

**⚠️ KRİTİK UYARI**: Her kullanıcı talebinde önce CONTEXT DETECTION yapılması ZORUNLUDUR. Bu adım atlanamaz!

Sistem her isteği analiz ederek otomatik olarak şu değişkenleri belirler:

### ANALİZ DEĞİŞKENLERİ
```javascript
CONTEXT = {
  PROJECT_TYPE: 'new' | 'existing' | 'debugging' | 'refactoring',
  COMPLEXITY: 'simple' | 'moderate' | 'complex',
  USER_INTENT: 'implementation' | 'learning' | 'fix' | 'optimize',
  FILE_COUNT: 1 | 'few' | 'many',
  FILE_SIZE_STATUS: 'safe' | 'approaching' | 'violation'
}
```

### DİNAMİK KURAL SEÇİMİ
```javascript
if (CONTEXT.FILE_SIZE_STATUS === 'violation') {
  LOAD: ['file-size-enforcement.mdc']
  BLOCK: all_new_code_additions
}
else if (CONTEXT.PROJECT_TYPE === 'new' && CONTEXT.COMPLEXITY === 'simple') {
  LOAD: ['clean-implementation.mdc']
}
else if (CONTEXT.USER_INTENT === 'learning' && CONTEXT.COMPLEXITY === 'complex') {
  LOAD: ['educational-approach.mdc', 'step-by-step.mdc']
}
// ... diğer kombinasyonlar
```

## COMPONENT DOSYALARI - MODÜLER YAPI

### 📋 Component Listesi
1. **`language-rules.mdc`** - Türkçe/İngilizce kullanım kuralları
2. **`file-size-enforcement.mdc`** - Dosya boyutu kontrolü (EN KRİTİK)
3. **`clean-code-standards.mdc`** - Kod kalitesi standartları
4. **`simple-task-handling.mdc`** - Basit görevler için kurallar
5. **`complex-project-management.mdc`** - Karmaşık projeler için kurallar
6. **`codebase-analysis.mdc`** - Kod tabanı analiz kuralları
7. **`emergency-protocols.mdc`** - Acil durum prosedürleri
8. **`educational-approach.mdc`** - Öğretim odaklı yaklaşım
9. **`mcp-tool-usage.mdc`** - MCP araç kullanım kuralları
10. **`quality-gates.mdc`** - Kalite kontrol standartları

### 📁 DOSYA YAPISI
```
.cursor/
├── rules.mdc (ANA MERKEZ - BU DOSYA)
└── components/
    ├── core/
    │   ├── language-rules.mdc
    │   ├── file-size-enforcement.mdc (KRİTİK)
    │   └── clean-code-standards.mdc
    ├── task-handling/
    │   ├── simple-task-handling.mdc
    │   ├── complex-project-management.mdc
    │   └── codebase-analysis.mdc
    ├── protocols/
    │   ├── emergency-protocols.mdc
    │   └── mcp-tool-usage.mdc
    └── specialized/
        ├── educational-approach.mdc
        └── quality-gates.mdc
```

## KURAL SETLERİ - DURUM BAZLI YÜKLEME

### SET_001: BASİT GÖREV (Tek dosya, basit değişiklik)
**Yüklenen componentler:**
- `language-rules.mdc`
- `file-size-enforcement.mdc` (MUTLAK)
- `clean-code-standards.mdc`
- `simple-task-handling.mdc`

### SET_002: KARMAŞIK PROJE (Çoklu dosya, yeni özellik)
**Yüklenen componentler:**
- `language-rules.mdc`
- `file-size-enforcement.mdc` (MUTLAK)
- `clean-code-standards.mdc`
- `complex-project-management.mdc`
- `codebase-analysis.mdc`
- `mcp-tool-usage.mdc`

### SET_003: ÖĞRETİM MODU (Kullanıcı öğrenmeye odaklanmış)
**Yüklenen componentler:**
- `language-rules.mdc`
- `educational-approach.mdc`
- `clean-code-standards.mdc`

### SET_004: ACİL DURUM (Kritik hata düzeltme)
**Yüklenen componentler:**
- `language-rules.mdc`
- `file-size-enforcement.mdc` (MUTLAK - hafif esneklik)
- `emergency-protocols.mdc`
- `clean-code-standards.mdc`

### SET_005: DOSYA BOYUTU İHLALİ (700+ satır tespit edildi)
**Yüklenen componentler:**
- `language-rules.mdc`
- `file-size-enforcement.mdc` (TAM BLOKAJ MODU)
- `codebase-analysis.mdc`

## KOD KALİTE KONTROLÜ - OTOMATİK SİSTEM

### 🚨 MUTLAK KONTROLLER (Her zaman aktif)
- [ ] Dosya boyutu limitleri (700-1000 satır)
- [ ] Kullanılmayan import temizliği
- [ ] Gereksiz kod eliminasyonu
- [ ] Türkçe iletişim standardı

### 📊 RESPONSE FORMATILARI
```
✅ Aktif kural seti: [SET_XXX]
🖥️ Sistem: Windows 11 PowerShell
🔒 GitHub: İzin kontrolü yapıldı
📏 Dosya boyutu durumu: [X satır / 1000 limit - DURUM]
🧹 Temizlik yapıldı: [liste]
🔄 Refactoring gerekli: [dosya listesi]
⚠️ Blokaj durumu: [sebep ve çözüm]
```

## COMPONENT YÜKLEME SİSTEMİ

### Örnek Senaryo:
```
Kullanıcı: "Yeni bir React komponenti eklemek istiyorum"

CONTEXT DETECTION:
- PROJECT_TYPE: existing
- COMPLEXITY: simple
- USER_INTENT: implementation
- FILE_COUNT: 1

KURAL SEÇİMİ: SET_001 (Basit Görev)

YÜKLENEN COMPONENTLER:
✅ language-rules.mdc
✅ file-size-enforcement.mdc
✅ clean-code-standards.mdc
✅ simple-task-handling.mdc

UYGULAMA:
1. Önce dosya boyutlarını kontrol et
2. Eğer 700+ satır varsa BLOKE ET
3. Değilse temiz implementasyon yap
4. Kalite kontrolü yap
```

## ÖZETİN ÖZETİ - CORE MESSAGe

**Bu sistem neden daha iyi:**
1. **Odaklanmış**: Sadece gerekli kurallar yüklenir
2. **Hızlı**: Cursor daha az metin işler
3. **Akıllı**: Durum bazlı otomatik seçim
4. **Modüler**: Component'ler bağımsız güncellenir
5. **Merkezi**: Tek noktadan yönetim
6. **Mutlak**: Kritik kurallar her zaman aktif

**EN ÖNEMLİ NOKTA:**
`file-size-enforcement.mdc` her zaman yüklenir çünkü dosya boyutu kontrolü MUTLAK kuraldır!

---

---
> Source: [xenitV1/cursor-dynamic-rules-system](https://github.com/xenitV1/cursor-dynamic-rules-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
