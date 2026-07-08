## lehrer-homepage

> This file provides context for AI assistants (e.g., Claude Code) working on this repository.

# CLAUDE.md — AI Assistant Guide for lehrer-homepage

This file provides context for AI assistants (e.g., Claude Code) working on this repository.

---

## Project Overview

**lehrer-homepage** is a static teacher homepage for Jan Herrmann at Oberschule Spelle (Lower Saxony, Germany). It consists of HTML pages, one CSS file, and JavaScript files — no build tools, frameworks, or package managers are used.

- **Language**: German (UI content and code comments are in German)
- **Tech stack**: HTML5, CSS3, Vanilla JavaScript (ES6+)
- **External services**: Google Fonts (CDN), Formspree (form submissions)

---

## Repository Structure

```
lehrer-homepage/
├── index.html           # Startseite: Bereiche-Übersicht, Was-ist-neu, Teaser, Kontakt
├── abgabe.html          # Student assignment submission form (Formspree-powered)
├── faecher.html         # Fächer-Übersicht (verlinkt jede fach-*.html)
├── fach-<fach>.html     # EINE öffentliche Seite pro Fach: Vorstellung + Material
│                        #   (deutsch, geschichte, wipo, informatik, werte-normen,
│                        #    mathe, englisch, sport, biologie, chemie, physik, musik,
│                        #    erdkunde, gestaltendes-werken, ag-projekte, faecheruebergreifend)
├── portal.html          # Login-geschützte Material-Schnellübersicht (Kolosseum-Redirect-Ziel)
├── kontakt.html · impressum.html · datenschutz.html
├── css/
│   └── style.css        # All site styles (inkl. globaler .mat-* / .material-karte / .fach-* Klassen)
├── js/
│   ├── main.js          # Navbar/Footer-Enhancer, Hamburger, Scroll-Spy, Footer-Jahr, Abgabe-Form
│   ├── homepage-gate.js # Auth-Gate: blendet geschützte Sektionen ein/aus
│   ├── kolosseum-login-widget.js # Login-Status-Chip in der Navbar
│   ├── kolosseum-prompt.js       # XP-Modal nach Quiz (Gast-Vorschau computeNotenpunkte)
│   ├── worksheet-xp.js           # XP-Modal/-Einlösung für Arbeitsblätter
│   ├── arena-bar.js              # Persistente Kolosseum-Statusleiste unten auf jeder Seite
│   ├── auth-guard.js             # Body verstecken bis Auth-Check (geschützte Einzelseiten)
│   ├── dynamic-content.js        # Lädt dynamische Inhalte (inhalte.json)
│   ├── materialien-dynamisch.js  # Lädt je Fach hochgeladene Materialien (Kolosseum-API, data-fach)
│   ├── was-ist-neu.js            # „Was ist neu?"-Spalten (neuigkeiten.json + inhalte.json + API)
│   ├── blog.js · blog-daten.js · blog-einreichen.js  # Blog-Rendering/-Fallback/-Upload
│   ├── supabase-config.js · supabase-leaderboard.js  # Globale Quiz-Bestenlisten
├── neuigkeiten.json     # Pflegbare Einträge für „Was ist neu?" (Funktionen)
├── inhalte.json         # Pflegbare Einträge für neue Materialien (dynamic-content.js)
├── materialien/         # Generierte interaktive Arbeitsblätter/Quizze (HTML)
│   └── _template-karten-quiz.html  # Vorlage für Bilderraten-/Karten-Quizze
├── ANLEITUNG-Bilderraten-Quiz.md   # Schritt-für-Schritt-Anleitung Bilderraten-Quiz
├── Material manuell von mir/  # Quelldateien (PPTX, Bilder, HTML-Entwürfe) von Jan Herrmann
├── upload/              # Drop folder for raw worksheet files
│   └── _erledigt/       # Processed originals (moved here after conversion)
├── regelradar.html      # RegelRadar (Rechtschreibung/Zeichensetzung): zentraler Hub
├── regelradar-selbsttest.html   # Schüler-Selbsttest (5 Varianten × 35 Items)
├── regelradar-lehrkraft.html    # Lehrkraft-Dashboard
├── regelradar-{diktat,strategien,strategieabfrage,uebungen,selbsttest2,quatsch}.html
├── reso*.html           # Reine Redirect-Stubs (alte Pfade → regelradar*.html)
├── reso-backend/        # RegelRadar-API-Backend (FastAPI + SQLite, systemd-Service)
│   ├── api.py           # Endpunkte /klasse, /klassen, /ergebnis, /ergebnisse, …
│   ├── reso-api.service # systemd-Servicedatei (Pfad bleibt /reso-api – Server-Vertrag!)
│   └── setup.sh         # Einmaliges Server-Setup-Skript
├── reso-deploy/         # Separater Deploy-Stand des Backends (eigene CLAUDE.md) – nicht das Live-Frontend
├── kolosseum/           # Lernkolosseum (geschützter Bereich, Node.js-Backend)
│   ├── public/
│   │   ├── profil.html · rangliste.html · charakter.html
│   │   ├── quiz.html · quiz-spiel.html · kampf.html · shop.html
│   │   ├── login.html · register.html
│   │   ├── js/ (main.js, arena-bar.js, avatar.js, charakter-avatar.js, site-nav.js)
│   │   └── admin/
│   │       ├── index.html · dashboard.html · quiz.html · schueler.html
│   │       ├── link-erstellen.html · material-hochladen.html · xp-vergabe.html
│   │       └── leseabenteuer.html
│   └── routes/
│       └── external.js  # XP-Berechnung (computeNotenpunkte)
└── tools/
    ├── ab_generator.py  # Watch-loop: upload/ → Claude API → materialien/ → git push
    └── requirements.txt # Python dependencies (anthropic)
```

> **Weitere eigenständige Lernseiten im Root** (öffentlich bzw. teils per `auth-guard.js`/Geheimlink
> geschützt), von den Fachseiten verlinkt: `narratologische-analyse.html`,
> `deutsch-dialektische-eroerterung.html`, `deutsch-das-parfum.html`, `deutsch-theaterprojekt.html`,
> `fuenfschrittlesemethode.html`, `stilmittel-quiz.html`, `literaturwissenschaft_quiz_v2.html`,
> `rechtschreibquiz.html`, `lernquiz_jahrgang5.html`, `ab-herrschaft-mittelalter.html`,
> `leseabenteuer.html` (+ `-regeln`, `-heldenbogen`), `kroatien-2026.html`, `wintersport-q8z2x.html`,
> `lehrer-upload.html`, `blog.html`, `blog-einreichen.html`, `kolosseum.html`.

### Manuell bereitgestellte Quelldateien (`Material manuell von mir/`)

Dieser Ordner enthält Dateien, die Jan Herrmann direkt hochgeladen hat (z. B. PowerPoint-Präsentationen, Bilder, HTML-Entwürfe). Er liegt im GitHub-Repository unter dem Pfad `Material manuell von mir/`.

**WICHTIG für Claude:** Bevor du bei einem Auftrag wie „Erstelle ein Arbeitsblatt / eine Präsentation / ein Lernmaterial zu Thema X" Inhalte aus dem Nichts generierst, **durchsuche zuerst diesen Ordner auf GitHub** (`mcp__github__get_file_contents` mit `path: "Material manuell von mir"`). Dort liegende Quelldateien (z. B. `.pptx`, `.pdf`, `.jpg`) sind als Vorlage zu verwenden und in das Zielformat (interaktives HTML in `materialien/`) zu überführen. Erst wenn dort nichts Passendes liegt, darfst du Inhalte eigenständig generieren.

---

## How to Run

No installation or build step is needed. Open files directly:

```bash
# Option 1: Open in browser directly
open index.html

# Option 2: Serve with Python
python3 -m http.server 8000

# Option 3: Serve with Node
npx http-server .
```

Visit `http://localhost:8000` to view the site.

---

## Globale Layout-Komponenten (verbindlich auf JEDER Seite)

### Header / Navbar

Die Navbar erscheint **auf jeder einzelnen Seite** – sowohl auf der Hauptdomain als auch im Kolosseum (Subdomain). **Keine Seite darf einen abweichenden Header haben.**

#### Design (Stand 2026-06): Schwebend + Icon-Navigation

Kopf- und Fußbereich sind **nicht durchgezogen**, sondern **schwebende „Karten"**
(abgerundet, Schatten, seitlicher Abstand, dunkler Blau-Verlauf). Die Navigation
zeigt **nur zentrierte Icons** (alle gleich hoch); der Linktext steckt in einer
Screenreader-only-`.nav-label` und erscheint beim Hover/Fokus als **Tooltip**
(`data-tooltip`). Oben links **und** unten links steht die **Bildmarke**
(`img/logo.svg` — Buch/Laptop mit JH-Monogramm, Navy/Gold).

#### Navbar-Elemente (von links nach rechts)

| Element | Icon | Ziel | CSS-Klasse |
|---|---|---|---|
| **Logo (Bildmarke)** | (Buch/Laptop) | `index.html` (Startseite) | `.navbar-logo` + `.navbar-logo-img` |
| **Was ist neu?** | ✨ | `index.html#was-ist-neu` | `.nav-item` |
| **Kontakt** | ✉️ | `https://lehrer-herrmann.de/kontakt.html` | `.nav-item` |
| *(Trenner „Geschützter Bereich")* | — | — | `.navbar-gb-trenner` + `.navbar-gb-label` |
| **Arena** | ⚔️ | `https://kolosseum.lehrer-herrmann.de/profil.html` | `.nav-item .navbar-gb-link` |
| **Blog** | ✍️ | `blog.html` | `.nav-item .navbar-gb-link` |
| **Fächer** | 📚 | Dropdown (`faecher.html` + Einzelfächer) | `.navbar-dropdown-toggle .nav-item .navbar-gb-link` |
| *(Login-Widget)* | — | — | `#kolosseum-widget` |

Jeder Icon-Link/-Button hat die Struktur:
`<span class="nav-icon" aria-hidden="true">ICON</span><span class="nav-label">Text</span>`
plus `class="nav-item"` und `data-tooltip="Text"`.

#### Verbindliches HTML-Snippet (Hauptdomain-Seiten)

```html
<nav class="navbar" role="navigation" aria-label="Hauptnavigation">
    <div class="container navbar-inner">

        <a href="index.html" class="navbar-logo" aria-label="Zur Startseite">
            <img src="img/logo.svg" alt="" class="navbar-logo-img" width="47" height="38">
            <span class="navbar-logo-text">lehrer-herrmann.de</span>
        </a>

        <button class="hamburger" id="hamburger" aria-label="Menü öffnen"
                aria-expanded="false" aria-controls="navbar-links">
            <span></span><span></span><span></span>
        </button>

        <ul class="navbar-links" id="navbar-links">
            <li><a href="index.html#was-ist-neu" class="nav-item" data-tooltip="Was ist neu?">
                <span class="nav-icon" aria-hidden="true">✨</span>
                <span class="nav-label">Was ist neu?</span></a></li>
            <li><a href="https://lehrer-herrmann.de/kontakt.html" class="nav-item" data-tooltip="Kontakt">
                <span class="nav-icon" aria-hidden="true">✉️</span>
                <span class="nav-label">Kontakt</span></a></li>
            <li class="navbar-gb-trenner" aria-hidden="true">
                <span class="navbar-gb-label">Geschützter Bereich</span>
            </li>
            <li><a href="https://kolosseum.lehrer-herrmann.de/profil.html" class="nav-item navbar-gb-link" data-tooltip="Arena – Lernkolosseum">
                <span class="nav-icon" aria-hidden="true">⚔️</span>
                <span class="nav-label">Arena</span></a></li>
            <li><a href="blog.html" class="nav-item navbar-gb-link" data-tooltip="Schüler*innenblog">
                <span class="nav-icon" aria-hidden="true">✍️</span>
                <span class="nav-label">Blog</span></a></li>
            <li class="navbar-dropdown">
                <button class="navbar-dropdown-toggle navbar-gb-link nav-item" aria-expanded="false" aria-haspopup="true" data-tooltip="Fächer" aria-label="Fächer">
                    <span class="nav-icon" aria-hidden="true">📚</span>
                    <span class="nav-label">Fächer</span>
                </button>
                <ul class="navbar-dropdown-menu" role="menu"><!-- Fächerliste --></ul>
            </li>
        </ul>

        <div id="kolosseum-widget" aria-label="Kolosseum-Anmeldung" style="display:none">
            <!-- Wird per js/kolosseum-login-widget.js befüllt -->
        </div>

    </div>
</nav>
```

> **Automatischer Umbau (`js/main.js`, Abschnitt 0):** Seiten mit *altem*
> Textmenü müssen nicht zwingend von Hand umgestellt werden – der
> `navbar-enhancer` in `js/main.js` rüstet jede Seite, die `js/main.js` +
> `css/style.css` lädt, **idempotent** auf Logo + Icon-Navigation + Footer-Logo
> um (überspringt bereits umgebaute Seiten). Für neue Seiten dennoch das obige
> Snippet verwenden (kein Aufblitzen, sauberes HTML).

Für **Unterverzeichnis-Seiten** (`materialien/`): Logo/Links als absolute Pfade (`/img/logo.svg`, `/index.html`, …).

Für **Kolosseum-Seiten** (`kolosseum.lehrer-herrmann.de`): alle Links als vollständige absolute URLs (`https://lehrer-herrmann.de/...`), Arena-Link als Relativ-Pfad (`/profil.html`), Logo als `https://lehrer-herrmann.de/img/logo.svg`. Der Umbau erfolgt dort über `kolosseum/public/js/main.js`.

#### CSS-Klassen der Navbar

| Klasse | Beschreibung |
|---|---|
| `.navbar-inner` | Schwebende Karte (Grid: Logo · zentrierte Icons · Widget) |
| `.navbar-logo` / `.navbar-logo-img` / `.navbar-logo-text` | Bildmarke + (auf Mobile ausgeblendeter) Wortmarken-Text |
| `.nav-item` | Einzelnes Icon (Link **oder** Dropdown-Button), gleiche Höhe |
| `.nav-icon` | Das sichtbare Emoji-Icon |
| `.nav-label` | Linktext, visuell versteckt (Screenreader); im Mobile-Menü sichtbar |
| `.nav-item[data-tooltip]` | Erzeugt den Hover/Fokus-Tooltip (zeigt das Linkziel) |
| `.navbar-gb-trenner` | Vertikaler Trennstrich (auf Mobile: horizontale Linie) |
| `.navbar-gb-label` | „GESCHÜTZTER BEREICH" – Desktop versteckt, Mobile sichtbar |
| `.navbar-gb-link` | Amber/goldene Linkfarbe für Arena, Blog, Fächer |
| `#kolo-widget .kolo-user-chip` | CSS-Override: gelber Text auf dunkler Navbar |

#### Erforderliche Scripts pro Seite

- Alle Seiten: `js/main.js` (Navbar/Footer-Umbau, Hamburger-Toggle, Footer-Jahr)
- Seiten mit Login-Widget: `js/kolosseum-login-widget.js`

### Footer

Der Footer erscheint ebenfalls **auf jeder Seite** als **schwebende Karte**
(`.footer-inner`). Er enthält:

- **Links:** Bildmarke (`.footer-logo` / `.footer-logo-img`) + `© [aktuelles Jahr] Jan Herrmann` (zusammengefasst in `.footer-marke`) | `Impressum` | `Datenschutz`
- **Rechts:** `Eingeloggt als [Avatarname], Rang [XP]` — und (nahezu unsichtbar) ein funktionierender Link zu `/kolosseum/public/admin/`, mit minimalem Kontrast. Wenn kein Nutzer eingeloggt ist: unsichtbar / leer.

Das Copyright-Jahr wird dynamisch via `id="footer-jahr"` gesetzt; das Footer-Logo
wird auf Alt-Seiten vom `navbar-enhancer` ergänzt (beides in `js/main.js`).

**Logo-Asset:** `img/logo.svg` ist eine Vektor-Nachbildung von „Logo 16"
(Buch + Laptop + JH, Navy `#1e3a5f` / Gold `#e0a32e`). Eine andere Datei kann
einfach unter gleichem Pfad ersetzt werden.

---

## Seitenstruktur (vollständig)

```
Startseite (index.html)
├── #ladebildschirm  → Kurze History-Timeline-Ladeanimation (blendet sich aus)
├── Aufbau-Banner    → (.aufbau-banner) Hinweis zwischen Navbar und Bereiche-Kacheln
├── #startseite      → Bereiche-Übersicht: 6 Kacheln in dieser Reihenfolge:
│                      1. Fächervorstellung (öffentlich, klickbar → faecher.html;
│                         Pill-Chips verlinken direkt auf die jeweilige fach-*.html)
│                      2. Arena – Lernkolosseum (🔒)
│                      3. Jan Herrmann – Wer bin ich? (öffentlich)
│                      4. Schüler*innenblog (🔒)
│                      5. Leseabenteuer (🔒)
│                      6. Materialien & Quizze (🔒)
├── #was-ist-neu     → Aktuelle Funktionen, Quiz-Leistungen, neue Materialien (js/was-ist-neu.js)
├── #gladiatoren-teaser → Kolosseum-Teaser mit Statistiken
├── #regelradar-teaser  → RegelRadar (Rechtschreibung & Zeichensetzung) → regelradar.html
├── Login-Gate       → sichtbar wenn NICHT eingeloggt (#homepage-login-gate)
├── #lernkolosseum   → Lernkolosseum-Teaser, sichtbar wenn eingeloggt
├── #digitale-materialien → Materialien-Grid mit Jahrgang-Filterleiste, sichtbar wenn eingeloggt
├── #blog-teaser     → Blog-Teaser, sichtbar wenn eingeloggt
├── #kontakt         → jan.herrmann AT obsspelle.de (E-Mail verschleiert)
└── Footer           → © Impressum · Datenschutz + versteckter Admin-Link

ÖFFENTLICH – Fächer (Grundprinzip: EINE Seite pro Fach = Vorstellung + Material)
├── faecher.html             → Fächer-Übersicht; jede Karte → eine fach-*.html
└── fach-<fach>.html         → öffentliche Fachseite: Fachvorstellung + Themen
                               + komplette Materialliste (Arbeitsblätter · Materialien · Quizze)
                               + dynamisch hochgeladene Materialien (.dyn-mat-section[data-fach])
    ├── Hauptfächer: fach-deutsch · fach-geschichte · fach-wipo · fach-informatik · fach-werte-normen
    ├── Weitere Fächer: fach-mathe · fach-englisch · fach-sport · fach-biologie · fach-chemie
    │                   · fach-physik · fach-musik · fach-erdkunde · fach-gestaltendes-werken
    ├── fach-ag-projekte           → AGs & Projekte
    └── fach-faecheruebergreifend  → fächerverbindende Quizze/Projekte
    (Die einzelnen Arbeitsblätter/Quizze in materialien/ bleiben hinter Login bzw.
     Geheimlink geschützt – nur die Material-LISTE ist öffentlich sichtbar.)

    ⚠️ Ehemalige materialien-<fach>.html sind ENTFERNT (waren login-geschützte Dubletten);
       ebenso fach-andere.html. Alle Verweise zeigen jetzt auf fach-<fach>.html.

GESCHLOSSEN (Login via Kolosseum-Account erforderlich)
├── portal.html  → login-geschützte Material-Schnellübersicht; verlinkt die fach-*.html.
│                  Ziel der Kolosseum-Redirects (login.html/register.html/students.js).
│                  Sekundär – die Fachseiten sind die kanonische Materialquelle.
├── Schüler*innenblog (blog.html)
│   └── blog-einreichen.html
└── Lernkolosseum
    ├── Übersicht/Landing (kolosseum.html) → auth-guard.js
    ├── Profil (kolosseum/public/profil.html) ← primäres Ziel aller „Zur Arena"-Links
    ├── Rangliste (kolosseum/public/rangliste.html)
    ├── Charakter · Quiz · Quiz-Spiel · Kampf · Shop (kolosseum/public/)
    └── Admin-Ebene (kolosseum/public/admin/) ← Zugang nur via verstecktem Footer-Link
        ├── index · dashboard · quiz · schueler · leseabenteuer
        ├── Einladungslink erstellen · Material hochladen · Manuelle XP-Vergabe
```

### Zugangslogik (`js/homepage-gate.js`)

- Prüft Kolosseum-Session via `GET /api/auth/me`
- **Eingeloggt:** Lernkolosseum-Teaser, Materialien-Sektion und Blog-Teaser werden eingeblendet; Login-Gate ausgeblendet
- **Nicht eingeloggt:** Login-Gate sichtbar; geschützte Sektionen ausgeblendet
- `#bereiche-uebersicht` ist **immer öffentlich** sichtbar, zeigt geschlossene Bereiche mit **„🔒 Login nötig"**

### Link-Ziel für Lernkolosseum

- Alle „Zur Arena"-Links (Bereiche-Übersicht, Lernkolosseum-Teaser, Navbar) → **`kolosseum/public/profil.html`**
- `kolosseum.html` ist sekundäre Übersichtsseite (via `auth-guard.js` geschützt)
- Öffentlich vorgelagerte Demo-Inhalte ohne Login: noch nicht vorhanden, werden später bewusst ergänzt

### XP-Berechnung: Externe Quizze (Notenpunkte-System)

XP werden nach dem Notenpunkte-System der gymnasialen Oberstufe vergeben:

**XP = Notenpunkte × Anzahl gespielter Fragen**

| Notenpunkte | Rohpunkte (%) |
|:-----------:|:-------------:|
| 15          | ≥ 95 %        |
| 14          | ≥ 90 %        |
| 13          | ≥ 85 %        |
| 12          | ≥ 80 %        |
| 11          | ≥ 75 %        |
| 10          | ≥ 70 %        |
| 9           | ≥ 65 %        |
| 8           | ≥ 60 %        |
| 7           | ≥ 55 %        |
| 6           | ≥ 50 %        |
| 5           | ≥ 45 %        |
| 4           | ≥ 40 %        |
| 3           | ≥ 33 %        |
| 2           | ≥ 27 %        |
| 1           | ≥ 20 %        |
| 0           | < 20 %        |

XP werden nur bei **Verbesserungen** gutgeschrieben (Differenz zum bisherigen Bestergebnis). Berechnung: serverseitig in `kolosseum/routes/external.js` (`computeNotenpunkte`), clientseitig (Gast-Vorschau) in `js/kolosseum-prompt.js`.

---

## Key Files and Their Roles

### `index.html`
- **Sektionsreihenfolge:** `#ladebildschirm` (Lade-Animation) → Navbar → Aufbau-Banner (`.aufbau-banner`) → Bereiche-Übersicht (`#startseite`, `.bereiche-uebersicht-section`) → `#was-ist-neu` → `#gladiatoren-teaser` → `#regelradar-teaser` → Login-Gate (`#homepage-login-gate`) → `#lernkolosseum` → `#digitale-materialien` → `#blog-teaser` → `#kontakt` → Footer
- **Bereiche-Grid:** 6 Kacheln in fester Reihenfolge:
  1. **Fächervorstellung** (öffentlich) — alle 15 Fächer als Pill-Chips; Klick auf Chip → Fachseite; Klick auf Karte → `faecher.html`; umgesetzt via `.bereich-karte-klickbar` + `onclick="if(!event.target.closest('a'))location.href='faecher.html'"`
  2. **Arena – Lernkolosseum** (🔒 Login)
  3. **Jan Herrmann – Wer bin ich?** (öffentlich)
  4. **Schüler\*innenblog** (🔒 Login)
  5. **Leseabenteuer** (🔒 Login)
  6. **Materialien & Quizze** (🔒 Login)
- Digitalematerialien-Sektion: Jahrgang-Filterleiste (`.dm-filter-leiste`) über dem Grid; Karten tragen `data-jahrgang="5-6|7-8|9-10"` Attribute
- `id="startseite"` sitzt auf der `<section class="bereiche-uebersicht-section">` (kein Hero mehr)
- Copyright year dynamisch via `id="footer-jahr"`
- Scripts: `main.js`, `dynamic-content.js`, `supabase-config.js`, `was-ist-neu.js`, `arena-bar.js`, `kolosseum-login-widget.js`, `homepage-gate.js` (+ Analytics/GoatCounter am Ende)

### Fächer: `faecher.html` + `fach-<fach>.html` (Grundprinzip)
- **Eine Seite pro Fach.** `fach-<fach>.html` ist die **öffentliche, kanonische** Fachseite und enthält Fachvorstellung **und** die komplette Materialliste (Kategorien Arbeitsblätter · Materialien · Quizze) plus `<div class="dyn-mat-section" data-fach="…">` für dynamisch hochgeladene Materialien.
- Die einzelnen Arbeitsblätter/Quizze in `materialien/` bleiben hinter Login bzw. Geheimlink geschützt – nur die **Liste** ist öffentlich.
- `faecher.html` ist die Übersicht (Navbar-Link „📚 Fächer"); jede Fachkarte verlinkt direkt auf die zugehörige `fach-<fach>.html`. Drei Gruppen: Hauptfächer · Weitere Fächer · AGs & Projekte.
- **Neue Materialien immer auf der `fach-<fach>.html` verlinken** (in der passenden Kategorie). Es gibt **keine** `materialien-<fach>.html` und **kein** `fach-andere.html` mehr.
- Scripts der Fachseiten: `main.js`, `materialien-dynamisch.js`, `dynamic-content.js`, `homepage-gate.js`, `arena-bar.js`. Fachseiten sind öffentlich (kein `auth-guard.js`, kein `noindex`).
- `portal.html` bleibt als login-geschützte Schnellübersicht bestehen (Kolosseum-Redirect-Ziel) und verlinkt ebenfalls die `fach-<fach>.html`.

### `abgabe.html`
- Student assignment upload form, Formspree-Integration
- `YOUR_FORM_ID` in Zeile ~68 durch echte Formspree-ID ersetzen
- `noindex, nofollow` — bewusst aus Suchmaschinen ausgeschlossen
- Akzeptierte Dateitypen: PDF, JPG, PNG, DOCX, ZIP (max. 10 MB)

### `js/main.js`
Vier eigenständige IIFE-Module:

| Modul | Zweck |
|-------|-------|
| Hamburger-Menü | Mobile-Nav-Toggle mit ARIA (`id="hamburger"` ↔ `id="navbar-links"`) |
| Scroll-Spy-Nav | Aktiven Nav-Link beim Scrollen hervorheben |
| Footer-Jahr | Aktuelles Jahr in `id="footer-jahr"` setzen |
| Abgabe-Formular | Validierung + Fetch-Submit (nur auf `abgabe.html`) |

### `js/kolosseum-login-widget.js`
- Zeigt Login-Status in `#kolosseum-widget` (wird von `id="kolosseum-widget"` zu `id="kolo-widget"` umbenannt)
- Eingeloggt → `.kolo-user-chip` mit Avatar-Emoji + Gladiatorenname (Link → Profil)
- Ausgeloggt → `.kolo-login-btn` „🏛️ Einloggen"
- Injiziert eigene `<style>` ins `<head>` — wird von `#kolo-widget .kolo-user-chip { color: #FFE66D !important }` in `style.css` überschrieben (Lesbarkeit auf dunkler Navbar)

### `css/style.css`
CSS Custom Properties (`:root`):
- `--farbe-primaer: #1e3a5f` (Dunkelblau)
- `--farbe-primaer-hell: #2d6a9f` (Mittleres Blau)
- `--farbe-akzent: #4a9eda` (Hellblau)
- `--farbe-hintergrund: #f4f6f9` (Seitenhintergrund)
- `--farbe-weiss`, `--farbe-text`, `--farbe-text-hell`, `--farbe-rahmen`, `--farbe-erfolg`
- `--schatten`, `--radius`, `--transition`

Responsive Breakpoints: `768px` (Tablet), `480px` (Mobil). Layout: CSS Grid + Flexbox.

Wichtige CSS-Klassen (ab 2026-05):
- `.navbar-gb-trenner` / `.navbar-gb-label` — Trenner + Beschriftung „Geschützter Bereich"
- `.navbar-gb-link` — amber/goldene Links (Arena, Blog, Fächer)
- `#kolo-widget .kolo-user-chip` — Farb-Override für eingeloggten Gladiatorennamen
- `.bereich-karte-klickbar` — `cursor: pointer` für Kacheln, die per `onclick` navigieren (ohne `<a>`-Wrapper)

**Hinweis Kolosseum-CSS** (`kolosseum/public/css/style.css`): Der Kolosseum-Server lädt seine eigene CSS-Datei, nicht die Haupt-`style.css`. Navbar-Stile sind daher dort separat mit hardcodierten Farbwerten dupliziert. Bei Navbar-CSS-Änderungen in `css/style.css` immer auch `kolosseum/public/css/style.css` am Ende (Abschnitt „SITE NAVBAR") aktualisieren.

---

## Coding Conventions

### HTML
- `lang="de"`, alle sichtbaren Texte auf Deutsch
- Semantische Elemente: `<nav>`, `<main>`, `<section>`, `<header>`, `<footer>`
- ARIA-Attribute auf interaktiven Elementen
- Icons: Unicode-Emoji (keine Icon-Fonts, keine SVGs)
- IDs: `kebab-case`, passend zu JavaScript-Selektoren

### CSS
- **Kein Präprozessor** — reines CSS
- CSS-Variablen für alle Wiederholungswerte
- Klassen: `kebab-case`, semantisch
- Section-Kommentare: `/* === ABSCHNITTSNAME === */`
- Keine Utility-Classes, kein CSS-Framework

### JavaScript
- **Kein Framework, kein npm** — Vanilla ES6+
- Jedes Feature: IIFE `(function() { ... })()`
- Keine globalen Variablen
- Benennung: Deutsch für fachliche Konzepte, Englisch für Code-Konstrukte
- Async: `fetch` mit `async/await`
- Fehlerbehandlung: `try/catch` mit deutschen Fehlermeldungen
- DOM-Zugriff: `querySelector` / `querySelectorAll`

---

## Development Workflow

### Änderungen vornehmen
1. Dateien direkt bearbeiten — kein Build-Schritt
2. Browser-Refresh zum Testen
3. Auf mehreren Viewport-Breiten testen: Desktop, Tablet (`768px`), Mobil (`480px`)

### Keine Tests oder Linting
- Kein Test-Suite, kein CI/CD, kein Linter
- HTML manuell oder mit W3C Validator prüfen
- JavaScript im Browser-DevTools-Console prüfen

### Git — PFLICHTREGELN FÜR CLAUDE

- **NIE direkt auf `main` committen oder pushen.** Kein einziger Commit darf direkt auf `main` landen.
- **Immer auf dem zugewiesenen Feature-Branch arbeiten.** Branch-Schema: `claude/<beschreibung>-<session-id>`. Falls noch nicht lokal vorhanden: `git checkout -b claude/<beschreibung>-<id>`.
- **Kein `git reset --hard main`**, kein Merge von `main` in den Feature-Branch ohne explizite Nutzeraufforderung.
- **Kein Force-Push auf `main`** — unter keinen Umständen.
- Nach Abschluss: Feature-Branch pushen (`git push -u origin <branch>`), dann den Nutzer fragen, ob gemergt werden soll. **Nie selbst mergen**, außer auf ausdrückliche Aufforderung.
- Commit-Befehl immer mit `-c user.email="jan@lehrer-herrmann.de" -c user.name="Jan Herrmann"`
- Commit-Nachrichten auf Deutsch oder Englisch

### Deploy — vollautomatisch via GitHub-Webhook

Der Server (`178.105.35.83`) zieht automatisch bei jedem Push auf `main`. **Kein manueller SSH-Befehl nötig.**

Einmalige Einrichtung (nur wenn Webhook noch nicht aktiv):
1. Auf dem Server in `/var/www/lehrer-homepage/kolosseum/.env` setzen:
   ```
   DEPLOY_SECRET=<zufälliges Secret>
   DEPLOY_DIR=/var/www/lehrer-homepage
   PM2_APP=kolosseum
   ```
2. In den GitHub-Repository-Einstellungen unter *Webhooks*:
   - Payload URL: `https://kolosseum.lehrer-herrmann.de/api/deploy`
   - Content type: `application/json`
   - Secret: dasselbe wie `DEPLOY_SECRET`
   - Event: *Just the push event*

### Server-Infrastruktur (Stand 2026-05)

**Webserver: Caddy** (kein nginx!) — Config unter `/etc/caddy/Caddyfile`:

```
lehrer-herrmann.de, www.lehrer-herrmann.de {
    root * /var/www/lehrer-homepage
    handle /reso-api/* {
        uri strip_prefix /reso-api
        reverse_proxy localhost:8400
    }
    file_server
}

kolosseum.lehrer-herrmann.de {
    reverse_proxy localhost:3000
}
```

Caddy neu laden nach Änderungen: `systemctl reload caddy`

**RegelRadar-API** (Service/Pfad bewusst weiterhin `reso-api`) läuft als systemd-Service auf Port `8400`:
- Service-Name: `reso-api`
- Prozess: `/opt/reso/venv/bin/uvicorn api:app --host 127.0.0.1 --port 8400`
- Datenbank: `/opt/reso/reso.db` (SQLite)
- Token: in `/etc/systemd/system/reso-api.service` unter `RESO_TOKEN=...`
- Neustart nach Token-Änderung: `systemctl daemon-reload && systemctl restart reso-api`
- Erreichbar von außen: `https://lehrer-herrmann.de/reso-api/`

**Kolosseum-Backend** (Node.js/PM2): läuft auf Port `3000`

---

## Formspree Setup

1. Account anlegen auf [formspree.io](https://formspree.io)
2. Neues Formular erstellen, Form-ID kopieren
3. In `abgabe.html` (Zeile ~68) ersetzen:
   ```html
   <!-- Vorher -->
   <form action="https://formspree.io/f/YOUR_FORM_ID" method="POST" ...>
   <!-- Nachher -->
   <form action="https://formspree.io/f/abcd1234" method="POST" ...>
   ```

---

## Accessibility Requirements

Alle Änderungen müssen sicherstellen:
- Semantische HTML-Struktur
- ARIA-Labels auf interaktiven Elementen
- Sichtbare Fokus-Zustände für Tastaturnavigation
- Ausreichender Farbkontrast (WCAG AA)
- Keine Information ausschließlich durch Farbe vermittelt

**Ausnahme (bewusst):** Der versteckte Admin-Link im Footer hat absichtlich minimalen Kontrast — das ist kein Accessibility-Fehler, sondern ein Feature.

---

## What NOT to Do

- Kein npm, kein Bundler (webpack/vite), kein CSS-Präprozessor einführen
- Kein JavaScript-Framework (React, Vue, Alpine etc.)
- Keine zusätzlichen Dateien anlegen, wenn nicht klar notwendig
- Keine ARIA-Attribute oder semantischen HTML-Elemente entfernen
- Keinen Text von Deutsch in eine andere Sprache ändern
- Keine globalen JavaScript-Variablen (IIFEs verwenden)
- Keine Farben hardcoden — bestehende CSS-Variablen verwenden
- Header oder Footer auf keiner Seite weglassen oder abweichend gestalten

---

## RegelRadar – Rechtschreibung & Zeichensetzung (Stand 2026-06)

**RegelRadar** (vormals „RESO") ist ein eigenständiges Diagnose- und Trainingssystem für den
Deutschunterricht. Claim: **„Rechtschreibung & Zeichensetzung diagnostizieren, trainieren, meistern."**

> **Namenskonvention:** Im **Frontend** durchgängig „RegelRadar"; „RESO" darf in sichtbaren Texten
> nicht mehr auftauchen. **Ausnahme (Infrastruktur):** Der Backend-API-Pfad `/reso-api`, der
> systemd-Service `reso-api` und das Verzeichnis `reso-backend/` behalten ihren Namen (Server-Vertrag).
> Die alten `reso*.html`-Pfade existieren nur noch als **Redirect-Stubs** → `regelradar*.html`.

### Seiten (alle im Root)
- **Hub:** `regelradar.html` – bündelt alle Unterbereiche, Hero + Claim, Logo `img/regelradar-logo.jpg`.
- **Schüler-Selbsttest:** `regelradar-selbsttest.html` – 5 Varianten × 35 Items, 7 Rechtschreibkategorien
  (Doppelkonsonanten, s/ß/ss, Auslautverhärtung, ä/äu, Zusammensetzungen, Ableitungen/Vorsilben,
  Groß-/Kleinschreibung). Klassencode + Name → Variante → MC-Test → Ergebnis → optional an Lehrkraft
  senden (POST `/reso-api/ergebnis`). API fest eingebaut: `https://lehrer-herrmann.de/reso-api`.
- **Lehrkraft-Dashboard:** `regelradar-lehrkraft.html` – Login via **Lehrkraft-Token** (Bearer-Header),
  4 Tabs: Ergebnisliste · Klassenheatmap · Schülerprofil · Klassen verwalten.
- **Weitere:** `regelradar-diktat.html`, `regelradar-strategien.html`, `regelradar-strategieabfrage.html`,
  `regelradar-uebungen.html`, `regelradar-selbsttest2.html`, `regelradar-quatsch.html`.
- **Teaser** auf der Startseite: `#regelradar-teaser` in `index.html` → `regelradar.html`.

### Backend (`reso-backend/api.py`) — Pfad/Service bewusst weiter „reso-api"
- FastAPI + SQLite, läuft als systemd-Service `reso-api` auf Port 8400
- Proxy via Caddy: `https://lehrer-herrmann.de/reso-api/` → `localhost:8400`
- **Wichtige Endpunkte:**

| Methode | Pfad | Auth | Zweck |
|---------|------|------|-------|
| POST | `/klasse` | ✅ Token | Neue Klasse anlegen |
| GET | `/klassen` | ✅ Token | Alle Klassen abrufen |
| POST | `/ergebnis` | ❌ öffentlich | Schülerergebnis einreichen |
| GET | `/ergebnisse/{code}` | ✅ Token | Ergebnisse einer Klasse |
| DELETE | `/ergebnis/{id}` | ✅ Token | Ergebnis löschen |
| DELETE | `/klasse/{code}` | ✅ Token | Klasse löschen |

### Token ändern (per SSH)
```bash
sed -i 's/RESO_TOKEN=alterToken/RESO_TOKEN=neuerToken/' /etc/systemd/system/reso-api.service
systemctl daemon-reload && systemctl restart reso-api
```

---

## Interaktive Arbeitsblätter (Worksheet-Generator)

Dieser Assistent unterstützt Jan Herrmann und sein Kollegium dabei, hochgeladene Arbeitsblätter (PDF, Bild oder Text) in interaktive, eigenständige HTML-Dateien umzuwandeln.

### Ablageort

Generierte Dateien → `materialien/`. Verlinkt werden sie auf der jeweiligen **`fach-<fach>.html`** (in der passenden Kategorie Arbeitsblätter · Materialien · Quizze); für die Startseiten-Übersicht ggf. zusätzlich in `inhalte.json`/`neuigkeiten.json` eintragen.

### Ausgabeformat

Jede Datei: vollständige standalone HTML-Datei mit eingebettetem CSS und JS, kein externes Stylesheet, responsiv (funktioniert auf Schüler-Smartphones).

**Ausnahme:** Worksheets, die auf `/css/style.css` referenzieren (ältere Dateien in `materialien/`), bekommen auch die Standard-Navbar (mit `/index.html`-Pfaden) und `main.js` + `kolosseum-login-widget.js` als Scripts.

### Designsystem für Arbeitsblätter

| Rolle             | Wert      | Verwendung                        |
|-------------------|-----------|-----------------------------------|
| Primärfarbe       | `#1e3a5f` | Überschriften, Buttons            |
| Akzentfarbe       | `#4a9eda` | Links, Fokus-Zustände             |
| Korrekt-Feedback  | `#2ecc71` | Grün bei richtiger Antwort        |
| Fehler-Feedback   | `#e74c3c` | Rot bei falscher Antwort          |
| Hintergrund       | `#f4f6f7` | Seiten-Hintergrund                |
| Karten-Hintergrund| `#ffffff`  | Aufgaben-Karten                   |

Weitere Vorgaben: `system-ui, sans-serif`, `border-radius: 8px`, `box-shadow` auf Karten, max. Breite `800px` zentriert.

### AB-Typen und ihre Umsetzung

| Typ | Umsetzung |
|-----|-----------|
| **Lückentext** | Input-Felder inline im Text, Auswertung per Button, Feedback pro Lücke + Gesamtpunktzahl |
| **Multiple Choice** | Radio/Checkboxen, klares Feedback nach Abgabe, kein Mehrfachversuch ohne Reset |
| **Zuordnung** | Drag & Drop oder Dropdowns je nach Komplexität |
| **Textanalyse / offen** | Textarea mit Zeichenzähler, Musterlösung aufklappbar (zunächst verborgen) |
| **Schreibaufgabe** | Strukturierte Textfelder mit Hilfestellungen, optionale Bewertungsrubrik |
| **Sonstiges** | Typ selbst erkennen, passendste interaktive Umsetzung wählen |

### XP, Bestenliste & Lösungsanzeige (verbindlich für ALLE Quizze)

Jedes Quiz (insbesondere die „stummen Karten" in `materialien/erdkunde_*`) muss folgende Mechanik enthalten – Vorlage: `materialien/erdkunde_niedersachsen-landkreise_jg5-10_2026-05.html` und `materialien/erdkunde_bundeslaender_jg5-10_2026-05.html`:

1. **XP-Vergabe** über `js/kolosseum-prompt.js` → `window.kolosseumReport(score, total, 'quiz-slug')` (Notenpunkte-System). Skripte am Dateiende einbinden: `/js/supabase-config.js`, `/js/supabase-leaderboard.js`, `/js/kolosseum-prompt.js`.
2. **Bestenliste** über `js/supabase-leaderboard.js` (`leaderboardSave` / `leaderboardFetch` / `leaderboardHTML`), Namenseingabe + globale Top-10 nach Abschluss.
3. **Lösungsanzeige umschaltbar:** Der Button „Lösung zeigen" darf die eigenen Eingaben **nicht dauerhaft verstecken**. Er ist ein **Toggle** zwischen „Lösung" und „Meine Antworten zeigen", sodass Lernende beliebig hin- und herklicken und ihre Treffer mit der Lösung vergleichen können.
4. **Faire XP-Wertung:** Sobald die Lösung **das erste Mal** aufgedeckt wird, wird die XP-würdige Punktzahl auf den Stand **vor** der Hilfe eingefroren (`lockedScore = currentCorrect()`). XP und Bestenlisten-Eintrag zählen nur das, was **ohne** Lösungshilfe richtig war; ein Hinweis dazu wird im Ergebnis angezeigt.

### Metadaten-Header

```html
<!--
  Titel: [Titel des AB]
  Fach: [erkanntes Fach]
  Klasse/Niveau: [erkanntes Niveau oder "nicht angegeben"]
  AB-Typ: [Lückentext / MC / Zuordnung / Offen / Gemischt]
  Erstellt: [Datum]
  Quelle: [Originalformat des hochgeladenen Dokuments]
-->
```

### Dateiname-Konvention

```
fach_thema_klasse_JJJJ-MM.html
```

Beispiel: `deutsch_einfuehrung_9r_2026-04.html`

### Workflow-Hinweis nach der Generierung

Nach der HTML-Ausgabe kurz angeben:
- Welche Inhalte nicht eindeutig erkannt werden konnten
- Ob Musterlösungen fehlen und nachgereicht werden sollten
- Ob Annahmen über das Niveau getroffen wurden

---
> Source: [LehrerHer/lehrer-homepage](https://github.com/LehrerHer/lehrer-homepage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
