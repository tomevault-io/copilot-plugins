## anitaschoenburg-de

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projekt-Übersicht

Statische Website für Anita Schönburg, Malerin. Kein Build-Prozess, kein Framework, kein Node.js. Alle Dateien werden direkt im Browser geöffnet oder per einfachem HTTP-Server bereitgestellt.

**Seiten:** `index.html` (Hauptseite), `galerie.html` (Galerie-Seite)

## Lokale Entwicklung

```bash
# Einfacher HTTP-Server (Python)
python -m http.server 8080

# Oder mit Node.js
npx serve .
```

## CSS-Architektur

Die Stile sind in fünf Dateien aufgeteilt, die in dieser Reihenfolge geladen werden:

1. `css/variables.css` — CSS-Custom-Properties (Farben, Schriften, Abstände, Größen)
2. `css/base.css` — Reset, globale Elemente (body, h1-h6, p, a, blockquote)
3. `css/layout.css` — Header, Navigation, Footer, Container, Sektionen
4. `css/components.css` — Button, Divider, Zitat, Lightbox, einzelne UI-Elemente
5. `css/gallery.css` — Galerie-Grid, Painting-Karte, Galerie-Vorschau

**Neue Stile immer in die thematisch passende Datei.** Keine neuen CSS-Dateien anlegen.

## JavaScript

`js/main.js` ist die einzige JS-Datei. Sie enthält:
- Jahresanzeige im Footer
- Mobile-Navigation (Hamburger-Toggle mit `is-open`-Klasse)
- Lightbox (`openLightbox(el)` / `closeLightbox(event)` — global, da via `onclick`-Attribut aufgerufen)

## Design-System (Brand Guide)

Die vollständige Design-Spec liegt in `docs/superpowers/specs/2026-03-29-brand-guide-design.md`.

**Farben (via CSS-Variablen):**
| Variable | HEX | Verwendung |
|---|---|---|
| `--color-pflaume` | `#521040` | Navigation, Überschriften, Buttons |
| `--color-gold` | `#D4AF37` | Akzente, Zierlinien, Hover — sparsam einsetzen |
| `--color-creme` | `#FAF8F5` | Seitenhintergrund — nie reines Weiß |
| `--color-text` | `#3A3A3A` | Fließtext — nie `#000000` |
| `--color-beige` | `#E8D5C0` | Platzhalter, sanfte Flächen |

**Schriften (Google Fonts):**
- `--font-heading` → `Great Vibes` — nur für Überschriften und kurze Zitate, **nie** für Fließtext
- `--font-body` → `Cormorant Garamond 300` — für alles Lesbare; kein `font-weight: bold`

**Ton:** Persönlich, direkt, „du"-Form. Keine Marketingsprache. Keine dritten Farben oder Schriften einführen.

## Bilder hinzufügen

Gemälde-Dateien gehören nach `assets/images/gemälde/`. Jedes neue Bild in `galerie.html` nach dem bestehenden Muster (`.painting`-Block mit `data-caption`, `onerror`-Fallback, `.painting-info`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MichaelSchoenburg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
