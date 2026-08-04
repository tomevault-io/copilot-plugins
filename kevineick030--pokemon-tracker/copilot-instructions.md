## pokemon-tracker

> Kevin ist Endnutzer mit ADHS und will keine Deployment-Überraschungen. Für JEDE Session (lokal wie Cloud):

# CLAUDE.md — Projekt-Statusdokument (Pokémon Karten Tracker)

## 🚦 Regeln für KI-Sessions (immer befolgen)
Kevin ist Endnutzer mit ADHS und will keine Deployment-Überraschungen. Für JEDE Session (lokal wie Cloud):

1. **Zuerst diese CLAUDE.md + START-HIER.md lesen.**
2. **Direkt auf `main` arbeiten — und die KI veröffentlicht selbst.** Wenn eine Änderung fertig & getestet ist, **committet und pusht die KI direkt nach `main`** (`git add -A && git commit -m "…" && git push`). Kevin muss dafür **keinen Ordner öffnen und keine `.bat` starten**. Push nach `main` = **live/produktiv** → danach in einem Satz sagen, was veröffentlicht wurde. **KEINE Preview-Branches oder Pull Requests**, außer Kevin fragt ausdrücklich danach. ⚠️ Die interaktive `deploy.bat` **nicht** selbst im Terminal ausführen (sie fragt nach Eingaben und bleibt hängen) — die ist nur Kevins Ein-Klick-Variante.
3. **Deployment:** Push nach `main` → die **GitHub Action** (`.github/workflows/deploy.yml`) macht `git pull` **und startet automatisch nur die Dienste neu** (`systemctl restart pokemon-tracker` + `pokemon-dashboard`). **Du musst nichts von Hand neu starten.** 🛑 **NIEMALS den ganzen Server rebooten** — geteilte VPS (mehrere Bots), das würde alle anderen killen; immer nur den einzelnen Dienst per `systemctl restart <name>`. Daten liegen in SQLite (`pokemon_tracker.db`).
4. **Doku aktuell halten (PFLICHT):** Nach JEDER nennenswerten Code-Änderung diese CLAUDE.md **im selben Commit** mit aktualisieren (neue Features, geänderte Abläufe, Deploy, Config). Veraltete Doku ist schlimmer als keine. Auch was Kevin als dauerhafte Regel sagt, kommt hierher – nicht nur in den Chat.
5. **Einfache Sprache, keine unnötigen Fachbegriffe.**

---

> Zweck: Damit eine **neue Session** sofort weiß, was das Projekt ist, was läuft,
> wie es betrieben/bedient wird und was noch offen ist. Bitte bei größeren
> Änderungen aktuell halten.

**Letzter Stand:** 2026-07-05 · Sprache mit dem User: **Deutsch** (Einsteiger,
nicht-technisch → einfache Schritt-für-Schritt-Erklärungen).

---

## 1. Was ist das?
Telegram-Bot + Web-Dashboard zum Tracken/Sammeln/Bewerten von Pokémon-Karten
**und versiegelten Produkten** (Tins, ETBs, Displays etc.).
Kernnutzung: **Karte/Produkt fotografieren → erkennen → Preis sehen → in
Sammlung/Watchlist legen.** Fokus: **deutsche Karten + Produkte, Cardmarket-Preise.**

---

## 2. Aktueller Stand (Ampel)

### 🟢 Funktioniert (live auf dem Server)

#### Foto-Workflow (Einzelkarten)
- Foto → Gemini-Erkennung (Name DE+EN+JP, Set, Nummer, Seltenheit, Produkttyp)
- Preis-Lookup-Kette:
  1. TCGdex → `idProduct` (Cardmarket-Produkt-ID) ermitteln
  2. **Lokaler Cardmarket Price Guide** (`cm_price_guide` SQLite-Tabelle) → `low`+`trend`+`avg7` (EUR)
  3. Fallback: TCGdex EU-Aggregate (wenn kein CM-Eintrag)
- Anzeige: `Ab: X €` (günstigstes Angebot EU-weit) + Trend + Ø7T + direkter CM-Link
- 4 Buttons: `[✅ Sammlung] [💰 Preis-Check] [🔔 Watchlist] [💼 Scalp-Track]`
  (Scalp-Track nur bei versiegelten Produkten sichtbar)
- Erkennungs-Info bleibt nach Button-Klick stehen (Buttons werden nur entfernt)

#### Cardmarket Price Guide (NEU, 2026-06-04)
- **Datei:** `https://downloads.s3.cardmarket.com/productCatalog/priceGuide/price_guide_6.json`
- **75.099 Pokémon-Produkte** mit low/trend/avg/avg7/avg30 (EUR)
- **Täglich 06:00** automatisch heruntergeladen + in SQLite (`cm_price_guide`) importiert
- Kein API-Key nötig. Lookup: `idProduct` → sofortige lokale DB-Abfrage (kein Rate-Limit)
- Modul: `cm_priceguide.py` (download_and_import, get_price, is_ready)

#### TCGdex-Lookup (zwei-Pfad-Strategie, NEU 2026-06-04)
- **Pfad 1 (bevorzugt):** Set + Kartennummer → direkter TCGdex-Endpunkt `/{lang}/cards/{set_id}-{num}`
  → immer exakt, kein Scoring-Fehler möglich. Plausibilitätscheck: Basis-Pokémon-Name.
- **Pfad 2 (Fallback):** Namenssuche → Nummer dominiert Scoring (+10 Treffer / -20 Mismatch).
  Wenn kein Kandidat die Nummer trifft → `None` statt falscher Karte.
- **Prinzip: lieber kein Preis als ein Preis von der falschen Karte.**

#### Sammlung (Einzelkarten + versiegelte Produkte)
- Foto → ✅ Sammlung → Kaufpreis eingeben → Eintrag in `portfolio`-Tabelle
- Funktioniert für Einzelkarten UND Tins/ETBs/Displays
- **Tägliche Neubewertung (06:10, direkt nach Price-Guide-Download):**
  `portfolio.update_all_values()` → TCGdex→CM-Price-Guide
- `/wert`, `/sammlung`, `/gekauft`

#### Japanische Karten
- Gemini-Prompt verlangt `card_name_en` als **PFLICHT** (auch bei JP-Karten, mit Beispielen)
- TCGdex EN-Lookup liefert `idProduct` → CM Price Guide-Lookup funktioniert
- Fallback: Namens-Suchlink wenn idProduct nicht gefunden

#### Weitere funktionsfähige Features
- **`/preis <name>`** (TCGdex-Preise), **KI-Chat** (Claude Haiku, Freitext)
- **Budget** (`/budget`, `/ausgabe`), **Watchlist** (`/add`, `/watchlist`)
- **Releases** (`/releases`, `/release_add`)
- **Web-Dashboard** (Sammlung-Galerie, Wert, Watchlist, Budget, Scalp, Releases)
  URL: `http://87.106.255.195:8090`

### 🟡 Eingeschränkt
- **Preise sind EU-weite Cardmarket-Durchschnitte** (kein DE-Filter): `low` = günstigstes
  EU-Angebot, nicht nur Deutschland. Der Link führt zu allen Cardmarket-Angeboten.
- **Versiegelte Produkte** (Tins/ETBs/Displays) haben **keine Preise** im CM Price Guide
  (der enthält nur Einzelkarten) → Preis zeigt `–`, manueller Kaufpreis beim Sammlung-Eintrag.
- Dashboard läuft nur über **HTTP** (Passwort, aber unverschlüsselt).
- (Watchlist-Alerts laufen inzwischen über den neuen Deal-Scanner, siehe 🟢 unten — nicht mehr über die blockierte Cardmarket-API.)

### 🟢 Schnäppchen-Scanner (neu gebaut 2026-06-13, Duplikat-Schutz 2026-07-05)
- **`deal_scanner.py`** (täglich 06:05, nach Price-Guide-Download): cacht SIR/IR-Karten per
  TCGdex-Rarity-Endpunkt, vergleicht `low` vs. `trend` aus dem CM Price Guide und schickt
  Deals als Telegram-Nachricht.
- **Duplikat-Schutz (Fix 2026-07-05 — vorher kam täglich die gleiche Liste!):**
  - Gemeldete Deals werden in Tabelle `deal_alerts_sent` gemerkt. Ein Deal kommt erst
    **wieder**, wenn sein Preis um ≥10 % **weiter gefallen** ist (oder 7 Tage vergangen sind).
  - **Keine neuen Deals = keine Nachricht** (Stille statt Wiederholung).
  - **Fake-Schnäppchen-Filter:** Angebote >70 % unter Trend werden ignoriert (fast immer
    beschädigte Karten oder Preisfehler, die ewig in der Liste kleben würden).
  - `/deals` zeigt weiterhin auf Abruf die komplette aktuelle Liste (ohne Filter).
  - Das 09:00-Briefing zeigt nur die **heute neu** gemeldeten Deals (aus `deal_alerts_sent`).
- **Watchlist-Alerts** laufen über `deal_scanner.check_watchlist_alerts`:
  erneuter Alert nur bei weiterem Preisrückgang (7-Tage-Gedächtnis in `alerts_sent`).
  Fehlende Cardmarket-Produkt-IDs werden beim täglichen Lauf automatisch nachgetragen.
- `deal_scorer.py` (Bewertung 0–100) wird derzeit **nicht** genutzt — bräuchte
  Verkäufer-/Zustandsdaten aus der blockierten Cardmarket-API. Liegt bereit.

### 🟢 Bugfixes 2026-07-05 (Code-Review)
- **💡 Kauf-Check-Button funktioniert jetzt** (stürzte vorher IMMER ab: Code verglich das
  Trend-Objekt statt des Marktpreises mit einer Zahl → TODO 1 „Kauf-Berater" ist damit live).
- **`/karte`-Buttons funktionieren jetzt** (Daten wurden vorher in falschem Format
  gespeichert → jeder Button-Klick lief ins Leere).
- **`/watchlist` + Dashboard-Watchlist zeigen echte Preise** aus dem CM Price Guide
  (lasen vorher aus der toten `price_history`-Tabelle → immer „–").

### 🔴 Läuft nicht / kaputt
- **Scalping** (retail_monitor/hotstock): Händler-Selektoren ungetestete Platzhalter,
  Sealed-Preise fehlen → keine echten Restock-Alerts. Die Module werden in main.py
  gar nicht mehr als Jobs eingeplant (nur noch die Commands existieren).
- **Cardmarket-API**: nicht verbunden (keine Tokens). Sobald 4 Tokens in `.env`,
  schaltet Bot auf echte DE-Filterung um (`config.cardmarket_enabled()`).
  ⚠️ Cardmarket vergibt aktuell keine neuen API-Zugänge → **echte „nur Deutschland"-Preise
  sind derzeit nicht möglich.** Workaround: alle Links filtern auf 🇩🇪-Verkäufer + NM,
  die angezeigten Zahlen (`low`/`trend`) bleiben aber EU-weit.

---

## 3. Deployment / Infrastruktur

- **Server:** Strato VPS, **IP `87.106.255.195`**, Ubuntu.
  ⚠️ **GETEILTER Server!** Dort laufen weitere Projekte:
  `sk-holzfabrik-bot`, `rookiecard-telegram-bot`, `max` (+ `max-admin`),
  `crypto-tracker`. **NIEMALS globale Befehle** (`pkill`, killall, globale
  Restarts) — nur `/opt/pokemon-tracker` und die eigenen systemd-Units anfassen.
- **SSH:** `ssh root@87.106.255.195` — passwortlos (SSH-Key auf Kevins PC).
  Claude kann direkt via `ssh root@87.106.255.195 "..."` Befehle ausführen.
- **Projektpfad auf dem Server:** `/opt/pokemon-tracker`
- **Dienste (systemd):**
  - `pokemon-tracker.service` → Bot (`venv/bin/python main.py`)
  - `pokemon-dashboard.service` → Dashboard (`gunicorn -w 2 -b 0.0.0.0:8090 dashboard.app:app`)
- **Dashboard-URL:** `http://87.106.255.195:8090` · Passwort: `.env` → `DASHBOARD_PASSWORD`

### Update-/Deploy-Workflow
1. Lokal ändern (`C:\Users\Kevin\Desktop\claude projekte\pokemon-tracker`)
2. `git commit` + `git push origin main`
3. Den Rest macht die **GitHub Action** (`.github/workflows/deploy.yml`) automatisch:
   `git pull` + `systemctl restart pokemon-tracker pokemon-dashboard`.
   Manueller Fallback nur falls die Action mal ausfällt:
   `ssh root@87.106.255.195 "cd /opt/pokemon-tracker && git pull && systemctl restart pokemon-tracker pokemon-dashboard"`
   (Nur diese Dienste neu starten — **nie** den ganzen Server.)

---

## 4. Architektur / wichtige Dateien

```
main.py              Einstieg: Bot + APScheduler-Jobs
bot.py               Telegram-Handler, Foto-Workflow, Buttons (on_callback)
config.py            .env laden (override=True!), Konstanten, cardmarket_enabled()
                     CM_PRICE_GUIDE_URL, CM_PRICE_GUIDE_DOWNLOAD_HOUR=6
database.py          SQLite-Schema + Zugriff (pokemon_tracker.db)
                     Tabelle cm_price_guide (NEU): 75k CM-Produktpreise
image_recognition.py Gemini (gemini-2.5-flash): card_name + card_name_en (PFLICHT)
                     + set_name + card_number + rarity + language + product_type
tcgdex.py            Name→idProduct-Mapper + Fallback-Preise
                     Zwei-Pfad-Suche: Set+Nr direkt (Pfad1) / Namenssuche strikt (Pfad2)
cm_priceguide.py     NEU: Cardmarket Price Guide Download + lokaler Lookup
                     download_and_import() / get_price(product_id) / is_ready()
pokeprice.py         ALT (pokemontcg.io) — nicht mehr genutzt, bleibt liegen
cardmarket.py        Cardmarket-API (nur aktiv wenn Tokens gesetzt, derzeit leer)
deal_scanner.py      Täglicher Schnäppchen-Scanner (TCGdex+CM Price Guide, 06:05)
                     + Watchlist-Alerts. Seit 07-05: get_new_deals() mit Duplikat-
                     Schutz (Tabelle deal_alerts_sent) + Fake-Schnäppchen-Filter
deal_scorer.py       Deal-Bewertung 0–100 — derzeit UNGENUTZT (bräuchte CM-API-Daten)
briefing.py          Baut die Texte fürs tägliche Briefing (09:00)
ai_chat.py           KI-Chat (Claude Haiku, Freitext-Fragen)
trend_analyzer.py    Preis-Trend-Analyse (steigend / fallend / stabil)
portfolio.py         Sammlungswert: TCGdex→CM-Price-Guide, tägl. 06:10
profit_calculator.py / scalp_targets.py /
retail_monitor.py / hotstock_monitor.py / restock_alerts.py /
release_calendar.py  → Scalping-Modul (teils nicht funktional)
dashboard/           Flask-Dashboard (app.py + templates/ + static/)
```

### Schlüssel-Entscheidungen
- **Preis-Architektur (2026-06-04):**
  - TCGdex: kostenlose API, liefert `idProduct` (Cardmarket-ID) + Fallback-Preise
  - CM Price Guide JSON: tägl. Download ohne API-Key, 75k Produkte, `low`+`trend`+`avg7`
  - Lookup-Kette: TCGdex→idProduct → lokale CM-DB → Fallback TCGdex direkt
- **TCGdex-Matching:** Kartennummer hat absoluten Vorrang. Falscher Preis ist schlimmer
  als kein Preis. Pfad1 (Set+Nr) schlägt immer Pfad2 (Name).
- **JP-Karten:** Gemini extrahiert immer englischen Namen → TCGdex EN-Lookup → idProduct OK.
- **`config.load_dotenv(override=True)`** — verhindert leere OS-Umgebungsvariable
  die `.env` überschreibt (Bug bei ANTHROPIC_API_KEY gehabt).
- **Gemini-Modell `gemini-2.5-flash`** (altes `gemini-2.0-flash-exp` → 404, abgeschaltet).
- **`.env`** enthält: Telegram-Token+ChatID, Anthropic-Key, Gemini-Key,
  Dashboard-Passwort+Secret, `CM_PRICE_GUIDE_URL`. Cardmarket-Tokens leer.
- **Geteilter Server** → niemals `pkill -f main.py` o.ä. — killt fremde Bots!

### Scheduler-Jobs (main.py) — Stand 2026-07-05
| Job | Wann | Status |
|-----|------|--------|
| CM Price Guide Download | 06:00 | 🟢 |
| Deal-Scanner + Watchlist-Alerts | 06:05 | 🟢 nur NEUE Deals (Duplikat-Schutz) |
| Portfolio-Bewertung | 06:10 | 🟢 direkt nach frischem Price Guide |
| Tägliches Briefing | 09:00 | 🟢 zeigt nur heute neu gemeldete Deals |
| Release-Kalender | 09:05 | 🟡 |

(Retail-/HotStock-/Sealed-Jobs sind NICHT mehr eingeplant — Module liegen ungenutzt im Repo.)

---

## 5. Offene TODOs / nächste Schritte

1. ✅ **Kauf-Berater (ERLEDIGT 2026-07-05):** 💡-Button nach Foto/Karte → Kaufpreis
   eingeben → KAUFEN/SKIP-Empfehlung. (War vorher durch Absturz-Bug komplett kaputt.)
2. ✅ **Sammlung lebt (ERLEDIGT):** Tägliche Bewertung via TCGdex+CM-Price-Guide.
3. ✅ **CM Price Guide (ERLEDIGT):** Tägl. Download, 75k Produkte, idProduct-Lookup.
4. ✅ **Strikteres Matching (ERLEDIGT):** Nummer dominiert, kein falscher Preis mehr.
5. **Sealed-Produkt-Sammlung:** Tins/ETBs/Displays per Foto in Sammlung aufnehmen
   (Scan funktioniert, ✅ Sammlung-Button vorhanden → testen ob es nun geht nach Bugfix).
6. **Sammlung-Extras:** Doppelte-Warnung beim Scannen, Set-Fortschritt (`8/18 SIR`),
   CSV/Excel-Export, Quick-Sell-Schätzung nach Gebühren.
7. ✅ **Watchlist-Scanner auf TCGdex/CM umgestellt (ERLEDIGT, `deal_scanner.py`)** → Schnäppchen- + Watchlist-Alerts wieder aktiv.
8. **Tote Scalping-Jobs aufräumen**: retail_monitor / hotstock_monitor / restock_alerts (Platzhalter-Selektoren).

---

## 6. Sicherheit / Gotchas
- **Secrets nur in `.env`** (gitignored). Niemals in `.py`/Templates.
- **Geteilter Server** → keine globalen Befehle (§3).
- **Telegram:** nur EINE Bot-Instanz pro Token gleichzeitig.
- Beim systemd-Neustart erscheinen „failed"-Meldungen wenn vorher Hintergrundinstanz
  per taskkill/pkill beendet wurde — das ist normal, kein Absturz.
- `CM_PRICE_GUIDE_URL` in `.env` falls Cardmarket den S3-Link ändert.

---
> Source: [kevineick030/pokemon-tracker](https://github.com/kevineick030/pokemon-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
