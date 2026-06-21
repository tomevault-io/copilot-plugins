## zahnarzt

> Projekt-Kontext für AI-Sessions. Kurz, dafür Pflicht-Lektüre.

# CLAUDE.md — Zahnärztehaus Arch (build/)

Projekt-Kontext für AI-Sessions. Kurz, dafür Pflicht-Lektüre.

Stand: 2026-05-16.

---

## Was diese Site ist

Statische HTML-Site für die Zahnarztpraxis Zahnärztehaus Arch (Bern, BE).
Deployment: **Cloudflare Pages** mit Pages Functions für `/api/contact` und `/api/reviews`.
Kein Build-Step (kein React, kein TS-Compile) — Edits gehen direkt auf live HTML/CSS/JS.

Live-URL (Preview): `https://buildv.pages.dev/`
Produktions-URL (geplant): `https://zahnaerztehaus-arch.ch/`

### Deploy-Status (2026-05-16)

Direct-Upload via `wrangler pages deploy . --project-name=buildv --branch=main --commit-dirty=true`
aus dem `build/`-Verzeichnis. Voraussetzung: `CLOUDFLARE_API_TOKEN` und
`CLOUDFLARE_ACCOUNT_ID` (`6b127e35ea82a4fcb54c290943506af8`) als Env-Vars gesetzt.

Pages-Projekt **`buildv`** hat im Dashboard bereits konfiguriert:
- KV-Binding `KV` → Namespace `1cbd1411e7454012a75a8a31bb7b51c0` (`zaa-arch-kv`)
- Env-Vars: `RECIPIENT_EMAIL=gasserseverin1@gmail.com` (Test-Empfänger),
  `SENDER_EMAIL=onboarding@resend.dev` (Test-Modus), `RESEND_API_KEY` und
  `TURNSTILE_SECRET` als verschlüsselte Secrets

Pipeline-Verify per `curl`: GET liefert `{"error":"method_not_allowed"}`,
POST mit Honeypot `{"error":"spam"}`, POST ohne Turnstile-Token
`{"error":"turnstile_failed"}`. End-to-end Mail-Versand am 2026-05-16
verifiziert: Browser-Submit auf buildv.pages.dev → Mail in Posteingang
(`gasserseverin1@gmail.com` als Test-Empfänger).

**Wichtig zum Test-Modus:** `onboarding@resend.dev` als Absender liefert
ausschliesslich an die Mail-Adresse, mit der der Resend-Account registriert
ist. Für Produktion muss `zahnaerztehaus-arch.ch` in Resend per DKIM/SPF/DMARC
verifiziert und `SENDER_EMAIL` auf `noreply@zahnaerztehaus-arch.ch` umgestellt
werden (siehe `functions/README.md` §6).

Live-Seiten (10):
- `index.html` (Startseite)
- `praxis/index.html`
- `behandlungen/index.html` (Redirect-Stub auf `/#behandlungen`)
- `behandlungen/{notfall,prophylaxe,zahnerhalt,zahnersatz,implantat,parodontose,kinder}/index.html`

Editorial-System dokumentiert in `_workshop/experiment-schnell/system/13-editorial-layout-system.md`
und Pattern-Library in `…/14-pattern-library.md`. Beide Files sind teilweise veraltet — Code
ist seither evolviert (siehe Abschnitt „Aktuelle Design-Reality" unten).

---

## Pflicht-Step nach JEDER HTML-Bearbeitung

```
node _workshop/scripts/validate-html.js
```

Muss `✓ Alle 10 Seiten OK.` zurückgeben, bevor die Session als „erledigt" gilt.

**Warum:** Am 2026-05-10 war `index.html` mid-statement abgeschnitten. Der gesamte
inline-`<script>`-Block (Mobile-Nav-Toggle, Footer-Year, Notfall-Logik, Sticky-Bar,
Form-Submit) lief nicht mehr — ohne dass Lighthouse, visuelles Review oder GSC
das aufgegriffen hätten. Erst der User hat es manuell gemerkt („Mobile-Nav öffnet
nicht"), und der Root Cause war 100 Zeilen entfernt.

Das Validation-Script:
1. Prüft, ob jede Datei mit `</html>` endet.
2. Zählt Tag-Paare (`<html>`/`<body>`/`<main>`/`<header>`/`<footer>`/`<script>`).
3. Extrahiert jeden inline-`<script>`-Block (skipped: `src=…`, `application/ld+json`)
   und parst ihn mit Node's `vm.Script`. Fängt unterminated strings, mid-statement
   EOF, fehlende Brackets, alle JS-SyntaxErrors.

Exit-Code 0 = OK, 1 = Fehler. Bei Git-basiertem Cloudflare-Pages-Build wäre das
Script als Build-Command konfiguriert (siehe `functions/README.md`). Bei **Direct
Upload** (aktueller Deploy-Modus) greift das nicht — vor Upload manuell ausführen.

---

## Aktuelle Design-Reality (Stand 2026-05-15)

Hat sich seit dem ursprünglichen Editorial-System mehrfach weiterentwickelt.
Verbindlich ist immer der Code in `assets/main.css` und `assets/subpages.css`.

### Farben (so wie sie wirklich verwendet werden)

- **Hero-Hintergrund:** Deep-Blue `#1B5BC0` mit komplexem 4-Layer-Overlay-Gradient
  (siehe `.hero--fullbleed .hero__overlay` in main.css — diagonal + vertikal + horizontal)
- **Sections-Background:** Off-White `rgba(250,250,249,1)` (default) und
  `var(--c-primary-bg)` für `.section--alt` (helles Brand-Blau)
- **CTA-Button:** Teal `#134e4a` (Hover `#0c3531`), Text Off-White `#fafafa`, 8px radius
  — gilt für `.btn--primary` in main.css UND subpages.css (heute synchronisiert)
- **Cards mit Hervorhebung:** `.triage-path` und `.treat-card` haben jetzt 24px
  border-radius (statt früherer 8px)
- **Warm-Tan-Akzent:** `#f0e6d3` für einzelne Closing-Highlights
- **Near-Black:** `#0d0d0d` Footer

### Typo

- Display = Serif (Heading-Klasse, gross + fett)
- Body = System-Font-Stack
- H1 im Hero: `clamp(3.2rem, 9vw, 7rem)`, font-weight 600, line-height 0.95

### Buttons

`.btn--primary` ist Teal mit Off-White-Text auf **allen 10 Seiten** (`main.css` und
`subpages.css` syncron). Der weisse Hero-Override (`.hero--fullbleed .btn--primary`)
für den Header-CTA auf dem Foto-Hintergrund bleibt invertiert (Weiss-Button mit
Teal-Text), weil sonst kein Kontrast gegen das Hero-Foto.

### Editorial-Layouts (aktiv verwendet)

- `.card-grid` und `.triage-flow` (3-Spalten-Cards)
- `.editorial-split` (5/7-Grid) — wurde aus dem Form-Bereich entfernt (Form ist jetzt
  zentriert via `.section-head--centered`)
- `.intro-spread__row` — drei alternierende Reihen, vertikal überlappend, mit weissem
  6px-Frame um jedes Bild. Reihen 1+2 (Haus, Wartezimmer) sind 16:10 quer mit
  8-Spalten-Layout, Reihe 3 (Daniela+Kind) ist 4:5 hochkant mit 5-Spalten-Layout.
- `.flow-closing` und `.editorial-pair` (7/5)

### Team-Bilder

- Rund (`border-radius: 50%`) mit 2px-Ring (`box-shadow: 0 0 0 2px rgba(11,36,68,.12)`)
- Lead-Bild (Daniela): aspect-ratio 1/1
- 5 kleinere Avatare: aspect-ratio 1/1

### Mobile-Navigation (komplexer als der Burger früher)

Drei Elemente nebeneinander rechts oben im Header (<768px):

1. **Praxis-Quick-Icon** (Haus-SVG) — öffnet eigenes Mini-Dropdown-Panel mit den 3
   Praxis-Submenu-Links (Über die Praxis, Räume & Sterilisation, Team)
2. **Behandlungen-Quick-Icon** (Zahn-SVG) — öffnet eigenes Mini-Dropdown mit den 7
   Behandlungs-Links
3. **Hamburger** — öffnet die volle Navigation als Dropdown unter dem Header.
   Submenüs darin sind **collapsed by default**; Tap auf den Parent-Link (Wort oder
   Caret) togglt das Submenu inline.

Die Quick-Panels werden via JS dynamisch generiert (Klon der `.nav__sub`-ULs aus der
Hauptnavigation, mit der `.nav__sub`-Klasse entfernt damit sie sichtbar werden) und
ans `<header>` angehängt. Click outside oder Escape schliesst alle Panels. Code: in
`assets/main.js` (Startseite) und `assets/subpages.js` (alle anderen Seiten),
byte-identisch dupliziert.

---

## CSS-Architektur (Stand 2026-05-15)

Refactor in 5 Schritten Mitte Mai 2026 — `main.css` und `subpages.css` haben jetzt
beide eine explizite Token-Schicht und die gleiche Cascade-Layer-Hierarchie. Beide
Stylesheets folgen der gleichen Konvention; nur Tokens-Werte unterscheiden sich
absichtlich (subpages hat wärmere Background-Töne `--c-bg:#fbf8f3` / `--c-bg-warm:#f0e6d3`).

### Cascade-Layer-Hierarchie

`main.css` deklariert in Z. 27:

```css
@layer reset, base, components, utilities, overrides;
```

Reihenfolge bedeutet Priorität — `overrides` schlägt `utilities`, `utilities`
schlägt `components`, usw. — **unabhängig von Spezifität oder Source-Order**.

Was wo liegt:

- **reset** — Universal-Reset (`*,*::before,*::after{box-sizing:border-box}`).
- **base** — Foundation: `:root` mit allen Tokens, `html`, `body`, `body::before`
  (Noise-Grain), `h1-h4`-Typografie, `p`/`a`/Underline-Regeln, Focus-Visible-
  Globalregeln, `img`/`svg`-Defaults, `.container`, `.skip-link`.
- **components** — Alle namentlichen Bausteine: Header, Nav, Button, Section, Hero,
  Trustbar, Triage, Treat-Grid, Team, Reviews, FAQ, Form, Footer, Sticky-Bar,
  Praxis-Strip, Editorial-Flow, Editorial-Overlap, Section-Divider. **Inklusive
  ihrer Component-Media-Queries** (z.B. `@media(max-width:880px){.team-lead{...}}`
  lebt im Components-Layer neben `.team-lead`).
- **utilities** — Helper-Klassen, die *gegen* Components drücken müssen:
  `.visually-hidden`, `.section-ornament`, `.dinkus`, `.section-rule`,
  `.has-dropcap`, `.marginalia`.
- **overrides** — Globale Modifier mit höchster Priorität: `@media print` und
  `@media (prefers-reduced-motion:reduce)`. Diese stehen am Datei-Ende.

**Regeln für neuen Code:**

- Neue Komponente → in `@layer components` einfügen. Die meisten neuen Stilen
  landen hier.
- Neuer Helper, der überall greifen soll → `@layer utilities`.
- Theme-Override für Premium/Handwerk-Varianten (siehe gasserwerk-design-system-
  Skill) → entweder eigene Layer zwischen `components` und `utilities`, oder
  in `overrides`. Die Layer-Liste ggf. erweitern.
- `!important` ist in Cascade-Layern **invertiert priorisiert**: `!important`
  in einer NIEDRIGEREN Layer schlägt `!important` in einer höheren. Daher
  defensiv nur dort einsetzen, wo unverzichtbar (`.visually-hidden`,
  `.form__honeypot`, `.footer-trust__sep`).
- Neue Cards/Akkordeons/Tiles bekommen `contain:layout style` — bei Components
  mit `overflow:hidden` + Image-Zoom auch `paint`. Spart Repaints beim Hero-
  Fixed-Background. Sticky Header mit Backdrop-Transition bekommt
  `will-change:background-color`.

Browser-Support: `@layer` ist seit Frühjahr 2022 in allen modernen Engines (Chrome
99+, Safari 15.4+, Firefox 97+). Sicher für die Site-Zielgruppe.

### Token-System (`:root` in `main.css` Z. 33–84)

Alle wiederkehrenden Werte sind als CSS-Custom-Properties zentralisiert. Hardcoded
Magic-Numbers im Code-Body sind in der Regel ein Bug — wenn ein Wert wiederholt
gebraucht wird, ist er als Token vorhanden:

- **Brand-Farben:** `--c-primary`, `--c-accent`, `--c-cta`, `--c-hero-blue`, …
- **Strukturfarben:** `--c-white`, `--c-footer-bg` (Near-Black), `--c-frame-warm`
  (Off-White für Bildrahmen)
- **Radien:** `--radius-sm` (4), `--radius-md` (8), `--radius-lg` (12),
  `--radius-xl` (24, für Cards mit Hervorhebung), `--radius-pill` (999, für
  Buttons/Badges), `--radius-circle` (50%, für Avatars)
- **Shadows:** `--shadow-sm/md/lg` + `--shadow-ring-frame` (subtiler 2px-Brand-Ring
  um Foto-Avatars)
- **Material-Texturen:** `--noise-grain` (feines globales Rauschen, `body::before`),
  `--noise-linen` (gerichtete Leinen-Webe, geteilt von Trustbar + Flow-Closing)
- **Typografie:** `--font-display` (Iowan/Palatino Serif), `--font-body`
  (system-ui first, dann -apple-system Stack)
- **Transition-Timing:** `--ease-fast` (.2s, Color/BG-Hover), `--ease-card`
  (.25s, Card-Lift + Shadow), `--ease-slow` (.35s mit Material-Cubic-Bezier,
  FAQ-Reveal), `--ease-header` (.7s mit weicher Sinuskurve, Header-„Atmen")
- **Layout:** `--container` (1184px), `--header-h` (64), `--notfall-h` (40)
- **Editorial-Flow:** `--section-py-flow`, `--reading-measure`, `--body-leading-flow`,
  `--item-gap-flow`

### Geteilte Patterns

Statt einzelne Klassen-Definitionen zu duplizieren, gibt es konsolidierte
Selektor-Listen für Patterns, die mehrfach vorkommen:

- **Eyebrow-Pattern** (Z. ~173): Vier Klassen teilen sich eine Basis —
  `.section-head__eyebrow`, `.hero__eyebrow`, `.trustbar__eyebrow`, `.flow-eyebrow`.
  Standardisiert auf `.82rem · letter-spacing .14em · gap 16px`.
  `.intro-spread__eyebrow` ist absichtlich separat (kleiner, ohne Akzent-Strich).
- **Avatar-Pattern** (Z. ~435): `.team-lead__img` und `.team-avatar__img` teilen
  gemeinsame Basis (kreisrunde Mask, Warm-Tan-Fallback). `.review-card__avatar` ist
  separat (Initial-Letter, kein Ring).

Bei Änderungen an einem Pattern: **prüfen, ob die Änderung für alle Klassen der
Selektor-Liste gilt**, oder ob ein neuer Selektor-spezifischer Override gebraucht
wird (siehe `.hero__eyebrow,.flow-eyebrow{margin-bottom:24px}` als Beispiel).

### Breakpoints (kanonisches Set)

Beide Stylesheets verwenden 5 Breakpoints — neue Media-Queries sollten dieselben
Werte nutzen, damit das Set scharf bleibt:

- **560px** (max-width) — Phone-klein: Layout-Stack, Typo-Shrink
- **768px** (max-width) — Tablet: Nav-Drawer-Schwelle
- **880px** (max-width) — Editorial-Grid-Wechsel (Magazin-Layouts)
- **881px** (min-width) — Komplement zu 880, für Magazin-Spread-Grids
- **980px** (max-width) — Standard-Mobile-Schwelle (Hero, Nav-enger)
- **1480px** (min-width) — Wide-Marginalia

Vorher waren 10 verschiedene Werte verstreut (520, 600, 700, 780, 900, 1060,
zusätzlich zu den kanonischen). Konsolidiert am 2026-05-16.

### Pflicht nach CSS-Edits

Validate-Script läuft nur über HTML — bei reinen CSS-Änderungen formell nicht
erforderlich, in der Praxis trotzdem empfohlen (CSS-Edits triggern oft auch
HTML-Anpassungen). **Visueller Sanity-Check** über `index.html` zu zentralen
Komponenten (Hero, Trustbar, Team, Cards, FAQ, Form, Footer) ist Pflicht, da kein
automatisierter Visual-Regression-Test läuft.

### Cache-Buster (Pflicht nach CSS-/JS-Änderungen)

Assets werden mit `?v=N` versioniert geladen. Ohne Bump cached der Browser die
alte Version — neue Änderungen sind dann nicht sichtbar, auch nicht nach Reload.

Nach jeder CSS-/JS-Änderung den Counter bumpen:

- **main.css**: in `index.html` (`<link rel="stylesheet" href="/assets/main.css?v=N">`)
- **subpages.css**: in **allen 8 Subseiten** (`praxis/index.html` + 7 Behandlungs-
  Subseiten) — Cache-Buster muss überall identisch sein, sonst lädt der Browser
  inkonsistent
- **main.js**: in `index.html`
- **subpages.js**: in allen 8 Subseiten

Konvention: einfach hochzählen (v=62 → v=63 → v=64). Keine semantische Bedeutung.

---

## Manuelle Sync-Pflicht: Schema-Rating ↔ Trustbar

`index.html` enthält ein `aggregateRating` in der Dentist-Schema-LD (für Google
SERP-Sterne). Werte sind hardcodiert nach der Trustbar (Stand 2026-05-10:
4.9★ aus 18 Bewertungen). Bei jeder Trustbar-Anpassung **muss auch das
Schema** mitaktualisiert werden — sonst zeigt Google veraltete Sterne. Such-Position:
HTML-Kommentar `<!-- Schema.org Dentist — aggregateRating ist hardcodiert -->`.

Phase-3-Idee: per Cloudflare HTMLRewriter aus dem `/api/reviews`-KV-Cache
automatisch in die Schema-LD injizieren. Dann entfällt diese Sync-Pflicht.

---

## Mail-Versand: Resend (nicht mehr MailChannels)

`/api/contact` versendet Mails über **Resend** (https://resend.com). Bis Mitte 2024
lief das über MailChannels — das wurde ersetzt, weil MailChannels Free-Tier für
Cloudflare-Workers gestrichen hat.

Environment Variables im Cloudflare Pages-Projekt (alle als „Encrypted"):
- `RESEND_API_KEY` (Resend-API-Key, beginnt mit `re_`)
- `TURNSTILE_SECRET` (Cloudflare Turnstile Secret)
- `RECIPIENT_EMAIL` (Empfänger der Anfragen)
- `SENDER_EMAIL` (Default: `onboarding@resend.dev` für Test; nach Domain-Verifikation
  in Resend auf `noreply@zahnaerztehaus-arch.ch` umstellen)

KV-Namespace `zaa-arch-kv` (ID: `1cbd1411e7454012a75a8a31bb7b51c0`) ist als Variable
Name `KV` ans Pages-Projekt gebunden, wird für Rate-Limiting verwendet (5/h pro IP).

---

## Was AI NICHT machen soll

- **Keine erfundenen Trust-Signale.** Keine Reviews, Google-Sterne nur mit Quelle,
  keine Mitgliedschaften die nicht real sind. SSO Standesordnung Art. 6 gilt.
- **Kein Heilversprechen.** Keine Garantien, keine „der beste", kein „schmerzfrei".
- **Nichts in production HTML truncaten.** Wenn Edits an grossen Files nötig sind,
  immer mit präzisem `old_string`/`new_string` arbeiten und danach validate-html.js
  laufen lassen.
- **Keine Tracking-Cookies / Analytics ohne explizite Zustimmung.** Site ist
  cookie-frei und soll es bleiben.
- **Header-Markup nicht aus dem Sync laufen lassen.** Der Header ist in allen 9
  Production-HTML-Files byte-identisch dupliziert (kein Template-Engine). Bei
  Änderungen am Header IMMER über alle 9 Files mit identischem `old_string` editieren.

---

## Workshop-Verzeichnis

Alle Experimente, Audits, Prompts und das Pattern-System liegen in `_workshop/`.

**Öffentlich nicht erreichbar** seit 2026-05-16 dank Catch-All-Pages-Function
`functions/_workshop/[[path]].js`, die für jeden `/_workshop/*`-Request eine
inline-404-Seite (Status 404, `X-Robots-Tag: noindex,nofollow`) zurückgibt.
Functions haben Routing-Priorität vor Static-Assets — die echten Files liegen
zwar noch im Pages-Upload-Bucket (Wrangler-Direct-Upload kennt keinen
zuverlässigen `.assetsignore`), sind aber nicht mehr abrufbar. `_redirects`
hilft hier nicht (greift nur, wenn KEIN Static-File am Pfad existiert);
`robots.txt` enthält `Disallow: /_workshop/` als Crawler-Hinweis.

- `_workshop/scripts/` — Build-Scripts (Bilder-Optimization, Editorial-Cascade,
  HTML-Validation).
- `_workshop/audits/` — datierte Audits, Photographic-Idiom-Briefings.
- `_workshop/experiment-schnell/system/` — Methodology-Dokumentation.

Bei neuer Iteration: Audit-File mit Datum prefix (`2026-05-NN-was.md`) anlegen.

---
> Source: [GasserwerkSolutions/Zahnarzt](https://github.com/GasserwerkSolutions/Zahnarzt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-21 -->
