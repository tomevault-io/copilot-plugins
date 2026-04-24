## kessel-boilerplate

> CSS-First Theme-System mit Supabase Storage und dynamischen Fonts.


# Theme-System

Dieses Projekt verwendet ein CSS-First Theme-System mit Supabase als Single Source of Truth.

## Architektur

```
┌─────────────────┐      ┌─────────────────┐
│   TweakCN       │ ───▶ │  Theme Manager  │
│   (Export)      │      │  (/themes/mgr)  │
└─────────────────┘      └────────┬────────┘
                                  │ Upload
                                  ▼
                         ┌─────────────────┐
                         │    Supabase     │
                         │  Storage/Table  │
                         └────────┬────────┘
                                  │ Load
                                  ▼
┌─────────────────┐      ┌─────────────────┐
│   globals.css   │ ◀─── │ ThemeProvider   │
│   (@theme)      │      │ (data-theme)    │
└─────────────────┘      └─────────────────┘
```

---

## 1. Theme Storage (Supabase)

### Themes werden gespeichert in:
- **CSS:** Supabase Storage Bucket `themes` → `{theme-id}.css`
- **Metadaten:** Supabase Tabelle `public.themes` → id, name, description, dynamic_fonts

### Sicherheit
- **Kein direktes Schreiben in themes-Bucket vom Client** - Nur über geprüfte Server-Routen/Actions

### Theme-Provider Aufgabe
Der Provider macht NUR EINS:
```typescript
document.documentElement.setAttribute("data-theme", themeId)
```

### VERBOTEN im Provider:
- `root.style.setProperty("--variable", value)`
- `root.style.fontFamily = ...`
- Jede direkte DOM-Style-Manipulation

---

## 2. Font-System

### Font-Kette
```
next/font (layout.tsx) → --font-{name} → globals.css → body
```

### Font laden (layout.tsx)
```tsx
// src/app/layout.tsx
import { Inter, JetBrains_Mono } from "next/font/google"

const fontSans = Inter({
  subsets: ["latin"],
  variable: "--font-inter", // NUR .variable verwenden!
})

const fontMono = JetBrains_Mono({
  subsets: ["latin"],
  variable: "--font-mono",
})

export default function RootLayout({ children }) {
  return (
    <html className={`${fontSans.variable} ${fontMono.variable}`}>
      {/* ... */}
    </html>
  )
}
```

### KRITISCH: .className-Verbot

| ❌ VERBOTEN | ✅ ERLAUBT |
|-------------|------------|
| `fontSans.className` | `fontSans.variable` |
| `inter.className` | `inter.variable` |
| Auf `<html>` oder `<body>` | Als CSS-Variable registrieren |

**Warum?**
- `.className` setzt `font-family` direkt mit hoher Spezifität
- Das überschreibt unsere CSS-Token-basierte Steuerung
- `.variable` stellt nur CSS-Variablen bereit (`--font-inter`, etc.)

### Font referenzieren (globals.css)
```css
:root {
  --font-sans: var(--font-inter);
  --font-mono: var(--font-mono);
}

body {
  font-family: var(--font-sans);
}
```

### VERBOTEN in CSS:
```css
/* ❌ Rohe Fontnamen */
body { font-family: "Inter", sans-serif; }

/* ❌ Externe Font-Links */
@import url("https://fonts.googleapis.com/...");

/* ✅ Nur CSS-Variablen */
body { font-family: var(--font-sans); }
```

### Font-Registrierung
- **Keine zusätzlichen Webfonts ohne Eintrag im zentralen Font-System** (`src/lib/fonts/registry.ts`)
- Neue Fonts müssen registriert werden, bevor sie in Themes verwendet werden können

---

## 3. Dark Mode

### Architektur
-  setzt  auf - Theme-Selektoren: - CSS-Variablen wechseln automatisch
- Siehe vollständige Doku: 
### Light/Dark Farbstrategie (OKLCH)
- **Light Mode**: Farben brauchen **niedrige Lightness** (L ≤ 0.5) für Kontrast auf weißem BG
- **Dark Mode**: Farben brauchen **hohe Lightness** (L ≥ 0.6) für Kontrast auf dunklem BG
- **Hue** bleibt identisch über beide Modi
- **Chroma** im Dark Mode leicht höher (dunkle BGs entsättigen visuell)
- **Foreground** ist immer die Gegenfarbe: Light-FG = Weiß, Dark-FG = Dunkel

### Beispiel Theme-CSS
```css
/* Light Mode */
[data-theme="ocean"] {
  --background: oklch(0.98 0.01 230);
  --foreground: oklch(0.15 0.02 230);
  --primary: oklch(0.55 0.2 230);
  --success: oklch(0.45 0.16 145);       /* L ≤ 0.5 für Light */
  --success-foreground: oklch(1 0 0);    /* Weiß auf dunkler Farbe */
}

/* Dark Mode */
.dark[data-theme="ocean"] {
  --background: oklch(0.12 0.02 230);
  --foreground: oklch(0.95 0.01 230);
  --primary: oklch(0.65 0.18 230);
  --success: oklch(0.65 0.19 145);       /* L ≥ 0.6 für Dark */
  --success-foreground: oklch(0.15 0 0); /* Dunkel auf heller Farbe */
}
```

---

## 4. TweakCN Import Workflow

### Via Theme Manager UI (einziger Weg)
1. Exportiere für "Tailwind v4" in TweakCN
2. Öffne `/themes/manager` in der App
3. Klicke "Theme importieren" und füge CSS ein
4. Theme wird automatisch in Supabase gespeichert

### NIEMALS:
- TweakCN-Export direkt in `globals.css` einfügen
- Lokale Theme-Dateien erstellen (alle Themes in Supabase)
- CSS-Variablen in JavaScript definieren

---

## 5. Radius Underflow-Schutz

In `globals.css` verwenden wir `max()` für sichere Radius-Berechnung:

```css
@theme inline {
  --radius-sm: max(0px, calc(var(--radius) - 4px));
  --radius-md: max(0px, calc(var(--radius) - 2px));
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
}
```

Dies verhindert negative Werte bei Themes mit `--radius: 0`.

---

## 6. Dynamische Fonts

Wenn ein Theme einen Font benötigt, der nicht in `layout.tsx` registriert ist:

```typescript
// src/lib/fonts/dynamic-loader.ts
export async function loadDynamicFont(fontName: string): Promise<void> {
  // Font wird zur Runtime über Google Fonts API geladen
  // und als CSS-Variable registriert
}
```

### Font-Validierung
```typescript
// src/lib/fonts/registry.ts
export const REGISTERED_FONTS = {
  "Inter": "--font-inter",
  "JetBrains Mono": "--font-mono",
  // ...
}

export function isFontRegistered(fontName: string): boolean {
  return fontName in REGISTERED_FONTS
}
```

---

## 7. ESLint Enforcement

Die Regel `local/no-next-font-classname-on-root` erzwingt:
- Kein `.className` auf `<html>` oder `<body>`
- Nur `.variable` ist erlaubt

---

## Dateien im Projekt

| Datei | Zweck |
|-------|-------|
| `src/app/globals.css` | CSS-Variablen, @theme Block |
| `src/app/layout.tsx` | Font-Registrierung (next/font) |
| `src/lib/themes/theme-provider.tsx` | Theme-Switching |
| `src/lib/themes/storage.ts` | Supabase Theme Storage |
| `src/lib/fonts/` | Dynamische Font-Loader |

---

## Checkliste für neue Themes

- [ ] Theme via Theme Manager UI importiert
- [ ] Light + Dark Mode CSS vorhanden
- [ ] `.dark[data-theme="x"]` Selektor für Dark Mode
- [ ] Keine rohen Fontnamen (nur `var(--font-x)`)
- [ ] Fonts validiert oder dynamisch geladen
- [ ] Radius-Werte kompatibel mit `max(0px, ...)` Schutz
- [ ] Status-Colors (success, warning, info, neutral) vorhanden
- [ ] Light-Mode: Status-Farben haben Lightness ≤ 0.5
- [ ] Dark-Mode: Status-Farben haben Lightness ≥ 0.6
- [ ] Foreground-Farben sind Gegenfarbe (Light: Weiß, Dark: Dunkel)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phkoenig) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
