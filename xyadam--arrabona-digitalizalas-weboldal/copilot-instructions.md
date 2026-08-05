## arrabona-digitalizalas-weboldal

> Ez egy modern, reszponzív **multi-termékes marketing weboldal** az **Arrabona** által fejlesztett két szoftverre:

# CLAUDE.md - Arrabona Digitalizálási és AI Chat Platform

## Projekt Áttekintés

Ez egy modern, reszponzív **multi-termékes marketing weboldal** az **Arrabona** által fejlesztett két szoftverre:

1. **Catalog+ v1.2** - Professzionális könyvtári digitalizálási és import szoftver
2. **Libra AI** - AI alapú dokumentum keresési és chat interfész

**Weboldal célja**: Marketing és lead generálás mindkét szoftverhez
**Célközönség**: Könyvtár adminisztrátorok, könyvtárosok, döntéshozók
**Nyelvek**: Magyar (alapértelmezett) és Angol
**Dizájn**: Világos mód, sötétkék (#1e3a8a) akcentszínnel

---

## Szoftveres Szolgáltatások Leírása

### 1. CATALOG+ v1.2 - Könyvtári Digitalizálási Platform

**Fő célja**: Professzionális könyvtári katalógus digitalizálása, adatkezelése és export

**Mit nyújt a szoftver?**
- **Katalóguskártyák digitalizálása**: Scannelt képekből és PDF-ekből AI alapú OCR-rel
- **Külső adatbázis importálása**: Más könyvtári rendszerekből származó adatok átvétele
- **Intelligens szövegfelismerés**: GPT-5 alapú OCR technológiával pontos adatkinyerés
- **Adatkezelés és tisztítás**: Automatikus és manuális adatfeldolgozás
- **Fuzzy search**: Intelligens keresés az összes feldolgozott dokumentumon
- **Standard formátumok exportálása**: Excel és USMARC formátumok

**Munkafolyamat (3 lépés)**:
1. **Importálás és Digitalizálás**: Scannelt képek, PDF, külső adatbázis (Arrabona csapata végzi az OCR feldolgozást)
2. **Böngészés és Keresés**: Digitalizált katalógus megtekintése, fuzzy matching keresés
3. **Exportálás**: Excel és USMARC formátumok, könyvtári kezelőrendszerekbe integráció

**Technikai Architektúra**: PySide6 GUI | Azure OpenAI GPT-5 OCR | SQLite/Supabase DB | Fuzzy matching | Excel/USMARC export

**Árazási Modell**:
- Kis (1-10k cédula): 100 Ft/cédula
- Közepes (10k-50k): 90 Ft/cédula (POPULÁRIS)
- Nagy (50k+): 80 Ft/cédula
- Kliens licensz: 30.000 HUF (egyszeri)

---

### 2. LIBRA AI - Intelligens Dokumentum Chat és Keresési Platform

**Fő célja**: Dokumentumokkal való beszélgetés, AI alapú keresés és információ kinyerés

**Mit nyújt a szoftver?**
- **Dokumentum feltöltés**: Scannelt képek, PDF-ek, szövegfájlok
- **AI Chat interfész**: Természetes nyelvű kérdések feltételére válaszok az adatbázisból
- **Intelligens keresés**: Vector-alapú szemantikus keresés
- **Forráskövetés**: Pontosan melyik dokumentumból származik az információ
- **RAG technológia**: Retrieval-Augmented Generation (nem hallucináció alapú)

**Munkafolyamat (3 lépés)**:
1. **Dokumentum Feldolgozás**: Feltöltés → Szöveg kinyerés → Chunking (512-1024 token) → Embedding (Gemini-001) → Vector DB
2. **Chat Felület**: Libra AI chat interfész elérhetővé válik
3. **Intelligens Keresés és Válaszadás**: Kérdés vektorizálása → Top-K chunk keresés → AI válasz (Gemini 2.5 Flash) → Forráskövetés

**Technikai Architektúra**: FastAPI | Google Gemini 2.5 Flash | Gemini-Embedding-001 | Supabase pgvector | React/Vue frontend

**Árazási Modell**:
- Ingyenes (0-100 oldal): 0 Ft
- Starter (101-5k oldal): 80 Ft/oldal
- Pro (5k-50k): 70 Ft/oldal (POPULÁRIS)
- Enterprise (50k+): 60 Ft/oldal

---

## Weboldalas Fájlstruktúra

```
arrabona_digitalizalas_weboldal/
├── index.html              # HOMEPAGE - Termékkiválasztó (2 kártya)
├── web_ocr/index.html      # Catalog+ marketing oldal
│   ├── js/lang_ocr_hun.js, lang_ocr_eng.js
│   └── img/product_*.png
├── web_chat/index.html     # Libra AI marketing oldal
│   ├── js/lang_chat_hun.js, lang_chat_eng.js
│   └── img/product_*.png
├── css/style.css           # OSZTOTT STÍLUS - Mindhárom oldalhoz
├── js/main.js              # OSZTOTT JAVASCRIPT - URL alapú nyelvfájl betöltés
├── js/lang_home_hun.js, lang_home_eng.js
└── img/logo.png, ocr.png, ai_chat.png
```

**Fájlstruktúra Logikája**:
- **Root (/)**: Homepage - termékkiválasztó oldal
- **web_ocr/**: Catalog+ specifikus oldal + erőforrások
- **web_chat/**: Libra AI specifikus oldal + erőforrások
- **css/ + js/ (root)**: Osztott stílus és logika mindhárom oldalhoz
- **img/ (root)**: Megosztott képek (logó, termékthumbnailok)

---

## Oldal Szekciók

### Homepage (index.html)
**Szerkezet**: Header + Hero (redukált) + 2 termékkártya + Footer
- `.hero:has(.hero-home)` - Redukált padding (30px)
- `.hero-home .hero-title` - 2.5rem; `.hero-home .hero-subtitle` - 1.1rem
- `.products-grid` - 2 oszlopú layout

### Catalog+ oldal (web_ocr/index.html)
**Szerkezet**: Header → Hero → Features (4 kártya) → Workflow (3 lépés Swiper) → Pricing (4 csomag + kliens licensz 30k Ft) → Contact → Footer

### Libra AI oldal (web_chat/index.html)
**Szerkezet**: Header → Hero → Intro → Features (3-4 kártya) → Workflow (3 lépés Swiper) → Pricing (3 csomag: Free, Starter 80 Ft/old, Pro 70 Ft/old*) → Pricing Note (Magyarázat az árról) → Contact → Footer

**FONTOS**: AI provider nevek (Google Gemini, Gemini-Embedding-001) **nem jelennek meg** a weboldalon!

---

## CSS & JavaScript

### CSS (style.css) - Osztott stílusfájl

**Szekciók**: Reset → CSS Variables → Layout → Header → Hero → Components → Responsive Breakpoints

**Szín Rendszer**:
- `--primary-blue`: #1e3a8a | `--dark-blue`: #1e40af | `--light-blue`: #3b82f6 | `--accent-blue`: #60a5fa
- `--white`: #fff | `--light-gray`: #f8fafc | `--gray`: #64748b | `--dark-gray`: #334155 | `--border-gray`: #e2e8f0

**Tipográfia**: Segoe UI (system fonts) | Line-height: 1.6 | H1: 3.5rem (2.5rem home) | H2: 2.5rem | Container: max 1200px

**Responsive**: Desktop (1200px+) → Tablet (768-1200px) → Mobile (480-768px) → Small (<480px)

**Animációk**: AOS (800ms) | 3D transforms (0.5s) | Hover (0.3s)

### JavaScript (main.js) - Dinamikus logika

**URL-alapú nyelvfájl betöltés**: `/web_ocr/` → OCR fájlok | `/web_chat/` → Chat fájlok | Root → Home fájlok

**Fő funkciók**:
1. AOS inicializálása (800ms, ease-in-out)
2. Swiper carousel (workflow szekciók): 1 slide, centered, pagination, keyboard
3. `updateLanguage()`: `data-i18n` attribútumok frissítése az aktuális nyelvből
4. Nyelvváltó dropdown kezelése
5. Contact form: mailto: link építés (produktum-specifikus subject)

### Nyelvfájlok struktura

**js/lang_home_hun.js, js/lang_home_eng.js**: `LANG_HOME_HU/EN` objektum
- `hero` (cím, alcím, leírás) | `products` (kártyaszövegek) | `footer`

**web_ocr/js/lang_ocr_hun.js, lang_ocr_eng.js**: `LANG_OCR_HU/EN` objektum
- `hero` | `features` (4 kártya) | `workflow` (3 lépés) | `pricing` (4 csomag) | `contact`

**web_chat/js/lang_chat_hun.js, lang_chat_eng.js**: `LANG_CHAT_HU/EN` objektum
- `hero` | `intro` | `features` (3-4 kártya) | `workflow` (3 lépés) | `pricing` (3 csomag) | `pricing_note` | `contact`

---

## Fejlesztés & Módosítás

### Szöveg módosítás
1. Nyisd meg: `js/lang_[home|ocr|chat]_hun.js` + `_eng.js`
2. Keress a `data-i18n` értékre (pl. "hero.title")
3. Módosítsd mindkét nyelvben
4. Frissítsd az oldalt (F5)

### CSS módosítás
1. Nyisd meg: `css/style.css`
2. Keress az osztálynévre (pl. `.hero-title`)
3. Módosítsd a tulajdonságokat
4. Tesztelj responsive módban

### Képek cseréje
- **Homepage**: `img/ocr.png`, `img/ai_chat.png`, `img/logo.png`
- **OCR**: `web_ocr/img/product_main.png`, `product_1.png`, `product_2.png`, `product_3.png`
- **Chat**: `web_chat/img/` (ugyanez)
- **Optimalizálás**: < 500KB, WebP preferált

### Linkek
- Homepage → OCR: `href="web_ocr/index.html"` | Homepage → Chat: `href="web_chat/index.html"`
- Vissza linkek: mindkét produktum footer-jében root `index.html`-re

---

## QA Checklist

**Nyelvváltás**: Magyar ↔ Angol működik? Összes `data-i18n` frissül?
**Responsive**: Desktop (4 oszlop features) → Tablet (2 oszlop) → Mobile (1 oszlop)?
**Carousel**: Nyilak, pagination, keyboard, drag működik?
**Képek**: Összes betöltődik? DevTools Network: nincs 404?
**Form**: Kitöltés → mailto: megnyílik, subject/body helyesen? Form ürítve?
**Perf**: Lighthouse > 90? Load time < 3sec?

---

## Deployment

**Netlify** (ajánlott): GitHub repo → Auto deploy → Custom domain
**Vercel**: GitHub import → Auto PR deploy
**GitHub Pages**: Egyszerű static hosting
**FTP**: Manuális feltöltés (könyvtárstruktúra megtartása!)

---

## Best Practices & Troubleshooting

**Kód**: DRY elv | Semantic HTML | Logikus CSS szervezés | Leíró nevek (`.feature-card` nem `.card1`)
**Perf**: Lazy load (`loading="lazy"`) | Minify CSS/JS | WebP + fallback | CDN (AOS, Swiper)
**Maint**: Git branching (`feature/`, `bugfix/`) | Jó commit üzenetek | CLAUDE.md frissítve | Changelog
**Fájlok olvasása**: Fájlokat **MINDIG teljes méretükben** nyissuk meg, az egészet beolvasva! Ez az alapvetően fontos az alapvető kódértéshez és az ismétlésből adódó hibák elkerüléséhez.

**Problémák**:
- Nyelvváltás nem működik → Nyelvfájlok betöltödtek? `data-i18n` helyesek? Console error?
- Képek hiányoznak → Fájlnév (case-sensitive!) | Path (relatív/abszolút) | DevTools Network
- Carousel nem működik → Swiper CDN betöltődik? CSS előtt van?
- Form nem működik → Email kliens van? Teszteld: `<a href="mailto:test@test.com">`
- Reszponzív nem működik → F5 refresh (nem csak resize) | Viewport meta tag | Media queries OK?

---

## Dokumentációs Irányelvek

**Változások naplózása**: Ha új feature vagy átszervezés történik (NEM hibajavítás):
1. CLAUDE.md-be írjuk, de **KOMPAKT formában** (1-2 sor max per elem)
2. Meglévő szekciót **módosítjuk** (nem új szekciót adunk)
3. Exempel: `**Pricing**: Libra AI-hez Enterprise tier hozzáadva (60 Ft/oldal 50k+)`
4. Bővíthető, de **ne legyen túl sok** - tartsd értelmesen szervezettnek
5. Régi verzió: maradjon a verzió historyban (v1.0, v1.1, stb.)

---

**v1.0** (2025-10-16): Multi-termékes weboldal + 3 marketing oldal + Dinamikus nyelvváltás + Reszponzív

---

**Kész: CLAUDE.md - Optimalizált Dokumentáció**

---
> Source: [xyadam/arrabona_digitalizalas_weboldal](https://github.com/xyadam/arrabona_digitalizalas_weboldal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
