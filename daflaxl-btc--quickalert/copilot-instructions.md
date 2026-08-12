## quickalert

> Dies ist die offizielle Website für QuickAlert - ein magnetisches LED-Warnlicht für Fahrzeuge.

# QuickAlert Website - Cursor Rules

## Projekt-Übersicht
Dies ist die offizielle Website für QuickAlert - ein magnetisches LED-Warnlicht für Fahrzeuge.
- **Tech Stack**: Next.js 16, React 19, TypeScript, Tailwind CSS 3.4
- **Sprache**: Deutsch (de_DE)
- **Deployment**: Vercel
- **Domain**: quickalert.eu

## Allgemeine Regeln

### Sprache
- Antworte IMMER auf Deutsch
- Alle UI-Texte, Kommentare und Commit-Messages auf Deutsch
- Rechtschreibung beachten: "ß" statt "ss" wo korrekt (z.B. "Bußgeld", nicht "Bussgeld")

### Code-Stil
- TypeScript mit striktem Modus
- Funktionale React-Komponenten mit Hooks
- `'use client'` Direktive nur wenn nötig (Client-Side Interaktivität)
- Tailwind CSS für Styling - keine separaten CSS-Module
- Pfad-Alias `@/*` für Imports verwenden

### Dateistruktur
```
app/           → Next.js App Router Seiten
components/    → Wiederverwendbare React-Komponenten
public/        → Statische Assets (Bilder, PDFs)
brand-assets/  → Logo-Dateien (SVG, PDF)
```

## Design-System

### Farbschema
**Light Mode (Hero-Bereich):**
- Warm/Holzig: `#F5E6D3`, `#D4B896`, `#6B4E3D`
- Akzent: `#F5A623` (QuickAlert Orange)

**Light Mode (Content-Bereich):**
- Hintergrund: `bg-white`, `bg-zinc-50`
- Text: `text-zinc-900`, `text-zinc-600`
- Akzent: `text-orange-500`, `text-orange-600`

**Dark Mode:**
- Hintergrund: `bg-black`, `bg-zinc-900`, `bg-zinc-950`
- Text: `text-white`, `text-zinc-300`, `text-zinc-400`
- Akzent: `text-orange-500`, `text-red-500`

### Typografie
- Schriftarten: Inter (Body), Poppins (Headlines)
- Headlines: `font-black`, `tracking-tight`
- Body: `leading-relaxed`

### Komponenten-Patterns
- Karten: `rounded-3xl` oder `rounded-[2rem]`, `border`, `shadow-xl`
- Buttons: `rounded-xl` oder `rounded-2xl`, `font-bold`
- Hover-Effekte: `hover:-translate-y-2`, `hover:shadow-2xl`, `transition-all duration-300`

## Produkt-Informationen

### QuickAlert BASE (29€) — DE-Markt, KEIN V16
- Erfüllt ECE R65 Lichtparameter (laut IDIADA-Tests)
- Neodym-Magnetfuß
- IP54 Wasser- & Staubschutz (laut IDIADA)
- **3x AA Batterien (LR6, Alkaline)** (NICHT AAA, NICHT USB-C/Akku!)
- Kompaktes Handschuhfach-Design
- Modell CH-019, EAN 4270005258300
- ⚠️ Darf NICHT als „V16" oder „homologada" beworben werden (kein DGT-Listing)

### QuickAlert PRO (49€) — ES-Markt, V16 conectada
- Alle BASE Features
- Integriertes GPS-Modul
- Automatische Cloud-Alarmierung an DGT 3.0
- **12 Jahre eSIM** inklusive (Telefónica IoT-Vertrag)
- IDIADA PC26020115 (vom 12.02.2026)
- Modell CH-019, EAN 4270005258317

### Zertifizierungen (alle vom Hersteller Ningbo Chakesi)
- IDIADA PC26020115 (V16 Spanien) — nur PRO
- UCA-Zertifikat E1 61522401 (Fertigungskontrolle, Pflicht für DGT)
- DGT 3.0 Connection Test bestanden (26.06.2025)
- Telefónica IoT Telco-Vertrag (12 Jahre Konnektivität, NICHT 13!)
- CE/RED/RoHS/EMV (von PTC, China)
- Spanien: V16 conectada Pflicht ab 01.01.2026

### Pflicht-Specs
- Stromversorgung: 3× AA Alkaline LR6 (NICHT AAA!)
- IP-Klasse: IP54 (laut IDIADA — falls Hersteller höhere Klasse zertifiziert, Datenblatt prüfen)
- Wind-Stabilität: 180 Pa getestet (220 km/h Marketing-Claim ohne separaten Test → vermeiden)
- Sichtbarkeit: niemals konkrete km-Angaben ohne Beleg → „auch aus großer Entfernung sichtbar"
- Betriebstemperatur: −10 °C bis +50 °C
- Gewicht: ~250 g

## Dark Mode vs Light Mode

### Light Mode
- Modernes Tech-Design
- Bento-Grid Layout für Features
- Cleane Pricing-Cards
- Zertifikat-Style für Zulassungen

### Dark Mode
- Story-Telling Format
- Ausführliche Texte über das Warndreieck-Problem
- Statistiken und Fakten
- Emotionaler Ansatz

## Git Workflow
- Commit-Messages auf Deutsch
- Format: `Fix:`, `Feat:`, `Refactor:`, `Style:`
- Umlaute in Commit-Messages vermeiden (ä→ae, ö→oe, ü→ue, ß→ss)
- Nach Änderungen immer pushen wenn User es wünscht

## Wichtige Hinweise
- Hero-Bereich (Opening Page) hat spezielles Design - nicht ohne Absprache ändern
- Dark Mode Toggle wechselt zwischen zwei verschiedenen Content-Strukturen
- Bilder in `/public/` - keine externen URLs
- Vercel Deployment erfolgt automatisch bei Push zu `main`

---
> Source: [Daflaxl-BTC/QuickAlert](https://github.com/Daflaxl-BTC/QuickAlert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
