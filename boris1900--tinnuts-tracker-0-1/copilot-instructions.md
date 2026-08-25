## tinnuts-tracker-0-1

> PWA + Android APK für Patienten der Tinnituspraxis Seedorf in Ahrensburg.

# Tinnitus Tracker – Projektdokumentation

## Zweck
PWA + Android APK für Patienten der Tinnituspraxis Seedorf in Ahrensburg.
Patienten tracken täglich ihre Tinnitus-Intensität und können ihren Tinnitus-Sound matchen.

**Live-URL:** https://app.tinnituspraxis-seedorf.de
**GitHub:** https://github.com/Boris1900/tinnuts-tracker_0.1

**🎉 Play Store: v3.4 LIVE seit 30.07.2026** — https://play.google.com/store/apps/details?id=de.tinnituspraxis.tracker
Paketname `de.tinnituspraxis.tracker`, Organisationskonto Tinnituspraxis Seedorf. Freigabe diesmal sehr schnell (unter einem Tag). Details siehe Abschnitt "Play Store" weiter unten.

---

## Aktueller Stand (Cache v39 / App v3.4)
Alle drei Wege synchron live: PWA, GitHub-APK-Release, Play Store. Export/Import-Fix und ehrlicher Update-Hinweis bestätigt im Store.

Alle App-Logik in `TinnitusTracker_Seedorf.html` – Vanilla JS/HTML/CSS, kein Build-Schritt.
Service Worker: `sw.js` · PWA + Capacitor Android APK · localStorage für Datenpersistenz.

### Navigation (6 Views)

| Tab | View-ID | Beschreibung |
|-----|---------|--------------|
| Eintragen | `view-track` | Intensität 1–10, bis zu 3×/Tag |
| Verlauf | `view-chart` | Liniendiagramm, 7/30/Alles |
| Tagebuch | `view-log` | Alle Einträge nach Datum |
| Blog | `view-blog` | Praxis-Artikel |
| Tester | `view-tester` | Tinnitus-Sound-Generator |
| ⚙️ | `view-settings` | Einstellungen |

---

## Versionierungs-Regel (PFLICHT – NIEMALS VERGESSEN)

⚠️ **Claude muss das bei JEDER Änderung selbstständig erledigen – nicht auf Boris warten!**

Bei **jeder** Änderung die deployed wird:

1. Cache-Version in `sw.js` hochzählen: `tinnitus-tracker-v35` → `v36` + `// APP_VERSION: v3.0` → `v3.1`
2. `CURRENT_CACHE` in `TinnitusTracker_Seedorf.html` hochzählen (muss mit sw.js übereinstimmen)
3. App-Version in `TinnitusTracker_Seedorf.html` hochzählen – steht an **zwei Stellen** (Einstellungen + Footer)
4. `CLAUDE.md` aktualisieren: "Aktueller Stand (Cache vXX / App vX.X)"
5. Commit + Push
6. Bei APK: `.\build-android.ps1` → Android Studio → APK bauen → umbenennen → `gh release create` → `download.html` aktualisieren

**Wichtig:** CURRENT_CACHE in HTML muss immer mit CACHE in sw.js übereinstimmen.
**Achtung:** "Browserdaten löschen" in Chrome löscht auch localStorage. Nutzer NIE dazu anleiten.

---

## Android APK – Build-Workflow

🚨 **CLAUDE MUSS `.\build-android.ps1` IMMER SELBST STARTEN – BEVOR Boris in Android Studio baut!**
🚨 **Ohne build-android.ps1 landet die alte HTML-Version in der APK! Ist bereits 2× passiert (v2.5, v2.8)!**

1. Änderungen in `TinnitusTracker_Seedorf.html` + Versionsnummern hochzählen
2. **`.\build-android.ps1` ausführen** – kopiert HTML als `www/index.html` + cap sync ← CLAUDE MACHT DAS
3. Erst dann Boris sagen: „Jetzt Android Studio → Generate APKs"
4. APK liegt in: `android/app/build/outputs/apk/debug/app-debug.apk`
5. Boris sagt „fertig" → Claude: Umbenennen (Rename-Item!) → `gh release create` → `download.html` aktualisieren

**Neue Datei hinzugefügt?** → Auch in `build-android.ps1` eintragen!

### Update-Mechanismus (APK)
- Update-Button lädt `sw.js` von Live-URL, liest `APP_VERSION`
- Bei Update: zeigt APK-Download-Link → `github.com/.../releases/download/vX.X/TinnitusTracker-vX.X.apk`
- Nach Klick: Hinweis „Downloads-Ordner öffnen und installieren"
- Einträge/Einstellungen bleiben bei Update erhalten (localStorage wird nicht angefasst)

---

## Analyse-Regeln (PFLICHT – nach Fehler vom 24.05.2026 eingeführt)

⚠️ **Diese Regeln gelten bei jeder Code-Analyse – ohne Ausnahme.**

1. **Vollständig lesen vor jeder Aussage:** Keine Behauptung über das Verhalten einer Funktion, bevor sie komplett gelesen wurde. Nie aus der Mitte einer Funktion schlussfolgern.
2. **Vermutung ≠ Tatsache:** Unsichere Analyse immer kennzeichnen: „Ich sehe X im Code – ich muss noch Y prüfen." Nicht als Fakt präsentieren.
3. **Vor jedem Fix live testen lassen:** Boris bittet, das konkrete Szenario in der echten App zu testen und zu beschreiben was er sieht. Erst dann beheben. Boris kennt die laufende App besser als der Code-Leser.

## Session-Workflow

Am Ende jeder Session: CLAUDE.md aktualisieren. Neue Session starten mit: „Lies die CLAUDE.md und sag mir kurz wo wir stehen."

---

## Letzter Stand

**v3.1 (16.07.2026):** Impressum + Datenschutz im App-Footer ergänzt (Datenschutz → https://www.tinnituspraxis-seedorf.de/app-datenschutz, gemeinsame App-Datenschutzseite für Ohreninsel/Augenblick/Tracker). apk.html- und download.html-Fallback auf v3.1 gezogen. PWA gepusht + APK-Release v3.1 erstellt. Hintergrund: Vorbereitung für den Play Store (rechtlich sauber). Impressum bleibt Praxis-Impressum (korrekt).

v3.0 war der vorherige Stand. Tester-Toggles repariert (Rollback auf v2.4-Basis). APK-Update-Flow verbessert: Update-Button öffnet apk.html statt direkt github.com.

**Landingpage (index.html):** Alle Grünflächen auf Markengrün `#7ed957` umgestellt (war vorher dunkles `#5c7a5c`/`#3d5c3d`). Betrifft: Nav-Button, CTA-Buttons, Schritt-Nummern, CTA-Sektion, Footer-Streifen, Quentn-Formular.

### Zusätzliche Dateien im Repo (kein App-Bestandteil)
- `apk.html` – Auto-Download-Weiterleitung für APK-Updates (kein GitHub-App-Konflikt)
- `download.html` – APK-Download-Seite für Erstinstallation
- `tester.html` – **Standalone Tinnitus-Klang-Tester** (Vorschau: app.tinnituspraxis-seedorf.de/tester.html)
  → Wird als eigenes Projekt ausgelagert: `C:\Users\Boris\Projekte\TinnitusTester\`
  → Eigene Domain geplant: `tester.tinnituspraxis-seedorf.de`
  → Neues GitHub-Repo + DNS-Eintrag bei all-inkl nötig

---

## Play Store

**Stand 30.07.2026: ✅ v3.4 LIVE im Play Store** — https://play.google.com/store/apps/details?id=de.tinnituspraxis.tracker
Enthält den Export/Import-Fix und den korrigierten Update-Hinweis. Freigabe diesmal unter einem Tag (eingereicht 29.07. abends, live 30.07. morgens), deutlich schneller als die Erst-Einreichung (5 Tage, 24.07. → 29.07.) – vermutlich weil Updates zu bereits freigegebenen Apps grundsätzlich schneller geprüft werden (gleiches Muster wie bei der Ohreninsel).

### Technisch (erledigt)
- Upload-Keystore: `android/keystore/tracker-upload.jks`, Alias `tracker` (gitignored). Passwort liegt im Passwortmanager, **ohne Keystore + Passwort sind keine Updates mehr möglich** → Backup der `.jks`-Datei nicht vergessen.
- `android/keystore.properties` (gitignored) + `signingConfig` in `android/app/build.gradle` (gleiches Muster wie bei Ohreninsel/TinnitusMediApp).
- Signiertes AAB gebaut über `cd android && ./gradlew bundleRelease` → `android/app/build/outputs/bundle/release/app-release.aab`.
- Paketname `de.tinnituspraxis.tracker` über Android-Entwickler-Identitätsbestätigung registriert und von Google verifiziert (ADI-Snippet in `android/app/src/main/assets/adi-registration.properties`, Debug-APK zur Bestätigung hochgeladen).
- **Für künftige Updates:** `versionCode`/`versionName` in `build.gradle` hochzählen, `bundleRelease` neu bauen, neuen Release im Produktions-Track hochladen.

### Store-Eintrag (erledigt)
- Kategorie: **Gesundheit & Fitness** (nicht Medizin).
- Gesundheitsfunktionen-Erklärung: **Krankheits- und Beschwerdemanagement** (unter "Medizin" in Googles Formular) – bewusst gewählt, weil die App aktiv Symptomintensität trackt (anders als die Ohreninsel, die nur Wellness/Entspannung ist). Regionale Zusatzanforderungen: aktuell keine (Stand 24.07.2026, Google informiert bei Änderungen).
- Datensicherheit: "Keine Daten erhoben" (App speichert ausschließlich lokal, kein Server, kein Tracking).
- Zielgruppe: 18 und älter.
- Länder: Deutschland, Österreich, Schweiz (DACH) – bewusst nicht weltweit, da App nur auf Deutsch.
- Anmeldedaten/Anzeigen/Behörden-Apps/Finanzfunktionen: jeweils "nicht erforderlich"/"Nein".
- Datenschutz-URL: https://www.tinnituspraxis-seedorf.de/app-datenschutz
- Store-Texte (Kurz-/Vollbeschreibung): `store-texte-entwurf.md` im Projekt-Root.
- Bildmaterial in `03-Design/Store-Assets/`: App-Symbol (`icon-512.png` im Projekt-Root), `feature-graphic-1024x500.png`, sowie `screenshot-1-eintragen.png` bis `screenshot-4-tester.png` (1080×1920, aus echten Handy-Screenshots hochskaliert, auch für 7"/10"-Tablet wiederverwendet).

### Wichtig / Erfahrung aus der Erst-Einreichung
- **Während die Prüfung läuft: nichts am App-Paket oder Store-Eintrag ändern** (setzt die Prüf-Uhr zurück).
- **Prüfdauer real: 5 Tage** (24.07. → 29.07.). Googles Ankündigung „bis zu 48h" ist bei Erst-Einreichungen unrealistisch, bei Ohreninsel waren es sogar 8 Tage.
- **28.07. (Tag 4): Play-Console-Helpdesk auf Englisch angeschrieben**, mit Verweis auf den Ohreninsel-Präzedenzfall. Antwort noch am selben Tag (Support „Ruth"): reiner Standard-Textbaustein, keine Bestätigung einer manuellen Prüfung oder Beschleunigung – „steht in normaler Prüfung, bis zu 7 Tage, keine weitere Version einreichen, Meldung folgt automatisch". Einen Tag später war die App live.
- **Lehre:** Ob der Helpdesk-Kontakt wirklich beschleunigt oder nur zufällig kurz vor der ohnehin fälligen Freigabe kam, lässt sich aus zwei Fällen nicht sagen. Er kostet aber nichts. Für die Zukunft: **erst ab Tag 7 anschreiben**, vorher gibt es nur den Textbaustein.

### Update-Workflow (ab jetzt, für jede künftige Version)

Bei jeder inhaltlichen Änderung – **beide Vertriebswege bedienen, PWA/Sideload UND Play Store:**

1. Änderungen in `TinnitusTracker_Seedorf.html`.
2. Versionsnummern hochzählen – **alles synchron, gleiche Nummer überall** (z.B. alle auf "3.2"):
   - `sw.js`: Cache-Version (`tinnitus-tracker-vXX`)
   - `TinnitusTracker_Seedorf.html`: `CURRENT_CACHE` (muss mit sw.js übereinstimmen) + App-Version an 2 Stellen (Einstellungen + Footer)
   - `android/app/build.gradle`: `versionName` auf dieselbe Nummer setzen
3. `android/app/build.gradle`: `versionCode` **immer um 1 hochzählen** (rein interne Google-Zahl, nie identisch mit versionName, dem Nutzer nie sichtbar, muss bei jedem Play-Store-Upload strikt steigen)
4. `CLAUDE.md` aktualisieren
5. Commit + Push
6. Sideload-Weg: `.\build-android.ps1` → Android Studio → Debug-APK → umbenennen → `gh release create` → `download.html`/`apk.html` aktualisieren
7. Play-Store-Weg: `cd android && ./gradlew bundleRelease` → signiertes AAB → Play Console → Produktion → Neuen Release erstellen → AAB hochladen → Versionshinweise → einreichen

---

## Verknüpfte Projekte

- **Blogartikel** (`C:\Users\Boris\Projekte\Blogartikel\`) – dort wird ein Blogartikel zur Veröffentlichung der TinnitusTracker App geschrieben, für die Wix-Homepage. Blogartikel-Arbeit gehört NICHT in diesen Ordner.

---

## Offene Punkte / Ideen

🔴 hoch / 🟡 mittel / 🟢 niedrig

### ✅ Erledigt (29.07.2026): Update-Check-Bug behoben + native Update-UI ehrlich gemacht
Beim Prüfen der Idee "Update-Button für Play-Store-Build entfernen" fiel auf: `checkForUpdate()` verlinkte bei JEDER nativen Installation (Play Store UND Sideload) im Update-Fall auf die GitHub-APK (`apk.html`) – **derselbe Bug, den die Ohreninsel-App schon hatte** (LOG-028 dort). Das steckte bereits in der live veröffentlichten v3.1.

**Weiterer Fund beim Durchspielen der Szenarien:** Der Versionsvergleich (GitHub-Pages-`sw.js` vs. eingebackene `CURRENT_CACHE`) ist für native Installationen strukturell irreführend, weil GitHub Pages sofort beim Push aktualisiert wird, das Play-Store-Review aber bis zu 7 Tage hinterherhinken kann (siehe TinnitusTracker-Erst-Einreichung). In diesem Fenster hätte die App "Update verfügbar" gemeldet, obwohl der Play Store objektiv noch nichts Neues hatte – Nutzer wäre auf der Store-Seite ohne Update-Button gelandet, verwirrt.

**Finale Lösung:** Für native Installationen (Play Store UND Sideload, technisch nicht unterscheidbar ohne eigenes natives Plugin) komplett auf den Versionsvergleich verzichtet. Button zeigt dort direkt "🛒 Play Store öffnen", statischer Hinweistext "Play Store hält dich automatisch aktuell", kein Rätselraten mehr. Der echte Versionsvergleich bleibt nur für Browser/PWA bestehen (dort ist er ehrlich, kein Review-Verzug).

**Bewusste Konsequenz:** Die GitHub-APK bleibt wie in der Ohreninsel-CLAUDE.md explizit ein **Sideload-Rückfall**, kein gleichwertiger, sich selbst aktualisierender zweiter Weg – wird bei jedem Release weiter aktuell gehalten und veröffentlicht, aber ohne eigene In-App-Update-Schleife.

**Migrations-Hinweis ergänzt:** Wer noch eine alte Sideload-APK hat und über den Button zum Play Store wechselt, bekommt einen Android-Signaturkonflikt (unterschiedliche Signierschlüssel, Play Store lässt sich nicht drüberinstallieren). Kurzer Hinweistext in der App dazu: erst Backup exportieren, alte App deinstallieren, aus dem Play Store neu installieren, Backup importieren – exakt der Weg, den Boris beim Testen von Hand durchgespielt hat.

### ✅ Play-Store-Link im Repo überall ausgerollt (Stand 29.07.2026)

- `download.html` – Android-Kachel zeigt jetzt den Play-Store-Link statt APK-Download (Priorität 1, erledigt).
- `TinnitusTracker_Seedorf.html` – In-App-Update-Check zeigt bei nativer Installation jetzt „Play Store öffnen" statt GitHub-APK-Link (siehe "Update-Check-Bug behoben" oben).
- `apk.html` – **bewusst unverändert gelassen**, dient jetzt als reiner Sideload-Rückfall (siehe Entscheidung oben), keine In-App-Verlinkung mehr dorthin.
- Button „App herunterladen" (Install-Overlay) verlinkt weiterhin auf `download.html`, damit automatisch mit erledigt.

**Noch offen: Schritt 2 – Wix-Praxisseite + Quentn-Mailstrecke**
- Auf der Wix-Praxisseite die Stelle anpassen, an der die E-Mail-Double-Opt-in-Strecke zwischengeschaltet ist (Schritt 2 der Anmeldung).
- In **Quentn** die Folgemail anpassen: Android-Link zeigt aktuell auf `download.html` (jetzt zwar mit Play-Store-Link, aber ggf. Text/Anleitung in der Mail selbst noch auf APK-Sideload ausgelegt) – prüfen und auf Play Store umstellen. Die Mails liegen komplett in Quentn, nicht im Repo.
- ⚠️ Ungeklärt bleibt weiterhin: `alltagshelfer.tinnituspraxis-seedorf.de` ist eine **Subdomain** (eigenes Projekt `C:\Users\Boris\Projekte\Alltagshelfer\`, GitHub Pages), nicht Teil der Wix-Seite – verlinkt aber selbst schon auf `download.html` und ist damit indirekt mit erledigt.

- 🟡 **Download-Flow vereinheitlichen (nach Play-Store-Freigabe), analog zur Ohreninsel:** Ist-Zustand: `index.html` hat ein eingebettetes Quentn-Formular (`data-form-id="331"`, Host `t88of5.eu-5.quentn-site.com`) für die E-Mail-Anmeldung. Double-Opt-in-Mail + Folgemail mit Gerätelinks sind komplett in Quentn konfiguriert (nicht im Repo). Aktuell vermutlich zwei getrennte Links in der Mail: Android → `download.html` (direkter APK-Download + Installationsanleitung), iPhone → direkt die App-URL (App erkennt iOS selbst per JS und zeigt eine eingebaute "Zum Home-Bildschirm"-Anleitung). Ziel: verschlanken auf einen einzigen Link in der Mail, der auf eine gemeinsame Seite führt (iPhone-Anleitung + Play-Store-Link für Android statt APK-Sideload), Quentn-Mail entsprechend anpassen. Erst umsetzen, sobald Google die App bestätigt hat (Play-Store-Link wird erst dann gültig).
- 🔴 Verlauf-Tab: Verhalten bei nur 1–2 Einträgen/Tag prüfen (Diagramm, Auswertung)
- 🟡 PDF-Export im Querformat anlegen (Diagramm braucht Breite)
- 🟡 Wochenübersicht / Monatsdurchschnitte ausbauen (30-Tage-Schnitt schon vorhanden)
- 🟢 Backup-Erinnerung alle 4 Wochen (Backup-Funktion selbst ist schon drin)
- 🟢 Morgen/Abend-Vergleich
- 🟢 Tester-Tab verbessern – erst nach Bewerbung der App priorisieren

---
> Source: [Boris1900/tinnuts-tracker_0.1](https://github.com/Boris1900/tinnuts-tracker_0.1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
