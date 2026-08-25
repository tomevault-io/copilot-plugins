## rollingcoin

> Dieses Dokument enthält projektspezifische Konventionen und Hinweise für Claude Code.

# CLAUDE.md

Dieses Dokument enthält projektspezifische Konventionen und Hinweise für Claude Code.
Lies es **vor jeder Code-Änderung**.

---

## Projekt-Überblick

**Coin Rain** ist eine Android-App (Marketing-Gag), bei der Euro-Münzen vom oberen
Bildschirmrand fallen und sich physikalisch im hohlen Smartphone-Gehäuse verhalten.
Sie wird als wiederverwendbares Library-Modul gebaut, das in bestehende Apps
eingebunden werden kann, und ist auf spätere iOS-Portierung vorbereitet.

Die ausführliche Aufgabenbeschreibung steht in `CLAUDE_CODE_PROMPT.md`.

---

## Modulstruktur — strikt einhalten

```
:core      → Reines Kotlin. Keine Android-Imports. Kotlin-Multiplatform-fähig.
:coinrain  → Android-Library-Modul. Dünner Adapter (SurfaceView, Sensor, Audio).
:app       → Demo-Activity. Bindet nur :coinrain ein.
```

**Regel:** Wenn du in `:core` Code schreibst und ein `import android.*` nötig
erscheint, baust du die Abstraktion an der falschen Stelle. Stop und überlege neu.
Beispiele für korrekte Trennung:

- Physik-Engine: `:core` ✅
- Münz-Datenmodell, Coin-Stückelungen: `:core` ✅
- Sound-Synthesizer (gibt PCM-Float-Array zurück): `:core` ✅
- `AudioTrack`-Anbindung: `:coinrain` ✅
- `SensorManager`-Anbindung: `:coinrain` ✅
- `Canvas`-Rendering: `:coinrain` ✅ (Canvas ist Android, deshalb hier)
- `CoinRenderer`-Interface: `:coinrain` (weil es Canvas referenziert)

---

## Bewusste Designentscheidungen — NICHT "wegfixen"

Der Vorgänger ist an Kollisionsproblemen gescheitert. Die folgenden Entscheidungen
wurden getroffen, um diese Probleme zu vermeiden, und dürfen nicht "verbessert"
werden, ohne explizite Abstimmung:

1. **Kein Impuls-Transfer zwischen Münzen — dissipativer per-Münze-Clamp.**
   Münze ↔ Münze macht Positionskorrektur (Penetration auflösen) plus einen
   streng dissipativen Schritt: jede Münze kappt lokal ihre eigene Annäherungs-
   geschwindigkeit entlang der Kontaktnormalen. Kein Momentum fließt von Münze A
   zu Münze B. Wenn du denkst „eigentlich müsste ich hier Impulse übertragen" —
   nein. Massegewichtete UND 50/50-Impulsübertragung erzeugen beide einen stabilen
   ~200 px/s-Grenzzyklus wenn Münzen in eine Ecke gepresst werden. Messreihe und
   Begründung: `ARCHITECTURE.md` Punkt 1 und `docs/PLAN.md`.

2. **Restitution-Slop.** Unterhalb einer Geschwindigkeitsschwelle (z. B. 50 px/s)
   wird Restitution auf 0 geclampt. Sonst hüpfen Münzen am Boden ewig im
   Mikro-Bereich.

3. **Sleep-State pro Münze.** Münzen mit niedriger Geschwindigkeit/Beschleunigung
   für N Frames werden eingefroren. Aufwachen nur durch Schüttelimpuls,
   Kollision durch wache Münze, oder Schwerkraftänderung > Schwellwert.

4. **Fixed Timestep mit Akkumulator.** Niemals `dt = frameTime` direkt in die
   Physik geben. Die Sim läuft mit fester 60 Hz, der Renderer läuft frei.

5. **Keine Compose, keine externe Physik-Lib, keine Audio-Assets.** Wenn du
   versucht bist eine Lib reinzunehmen, lies den Prompt nochmal.

Diese Punkte sind in `ARCHITECTURE.md` für Nachfolger festzuhalten.

---

## Code-Konventionen

- **Kotlin Style:** Kotlin Coding Conventions (offiziell). 4 Spaces.
- **Keine Wildcard-Imports.**
- **Public API minimal halten.** Was nicht von außen gebraucht wird, ist `internal`
  oder `private`.
- **Datenklassen** für Werte, **Sealed Classes** für Zustände/Events.
- **Keine Coroutines im `:core`-Modul** für Physik — die Sim läuft synchron im
  Render-Thread (Determinismus ist kritisch für Reproduzierbarkeit der Tests).
- **Nullability:** Niemals `!!` in Produktions-Code. Wenn nötig, durch
  `requireNotNull()` mit aussagekräftiger Message ersetzen.

---

## Tests

- Physik-Tests sind **JVM-Unit-Tests** in `:core`, kein Android-Instrumentation.
- Der **Stapel-Ruhe-Test** ist der wichtigste: 20 Münzen fallen, nach 5 s muss
  jede Münze schlafen (`|v| < v_sleep`). Wenn dieser Test rot ist, ist das Projekt
  nicht in lieferbarem Zustand.
- Tests müssen **deterministisch** sein. Verwende einen festen `Random`-Seed.
- Vor jedem Commit: `./gradlew :core:test :coinrain:test` muss grün sein.

---

## Config-System

- **Quelle der Wahrheit:** `config/coinrain.json`.
- **Lade-Mechanismus:** Gradle-Task generiert eine Kotlin-Datei
  (`CoinRainConfig.kt`) in `:core` zur Buildzeit. Keine Laufzeit-I/O.
- **Schema-Änderung an der Config:** Wenn du Felder hinzufügst, ergänze:
  - JSON-Schema in `config/coinrain.schema.json`
  - Default-Werte mit Kommentar in `coinrain.json`
  - Doku-Eintrag in `README.md` unter "Config-Referenz"

**Nicht** zur Laufzeit aus `SharedPreferences` lesen. **Nicht** eine GUI dafür bauen.

---

## Render-Performance — Faustregeln

- Münzen werden **einmalig pro Stückelung** in eine `Bitmap` vorgerendert. Im
  Frame-Loop nur `Canvas.drawBitmap()` mit Rotation. Niemals Vektor-Primitive
  pro Frame.
- Ziel: stabile 60 fps mit bis zu 100 sichtbaren Münzen auf einem Mittelklasse-
  Gerät (z. B. Snapdragon 6xx).
- Wenn die Framerate einbricht: erst messen (Systrace/Perfetto), dann optimieren.
  Nicht raten.

---

## Audio-Performance

- Maximal 8 gleichzeitige Sounds. Älteste werden verworfen.
- `AudioTrack` im STREAM-Modus, ein einziger Track mit eigenem Mixer (nicht
  pro Sound einen Track).
- **Energie-Schwelle:** Aufprälle unterhalb der Schwelle erzeugen *gar keinen*
  Sound, nicht nur leise. Sonst entsteht ein Klick-Teppich am Boden.

---

## Münz-Renderer

- `CoinRenderer` ist das Interface. Aktuell ein `ProceduralCoinRenderer` als
  Implementierung.
- Ein `SvgCoinRenderer` ist für später vorgesehen, aber **nicht jetzt**
  implementieren. Die offiziellen EZB-Wertseiten-SVGs liegen vor; sie werden in
  einem späteren Schritt eingebaut.
- Renderer-Wahl erfolgt über Config-Flag `renderer: "procedural" | "svg"`.

---

## Git-Workflow

- Aktiver Branch: `coinrain-rewrite`. Nie direkt auf `main` committen.
- **Ein Commit pro abgeschlossenem Schritt** (Config-System, Physik, Sound, etc.).
  Keine Riesen-Commits.
- Commit-Messages auf Englisch, imperativ, knapp:
  - `Add fixed-timestep physics core with sleep state`
  - `Implement procedural coin sound synthesis`
  - `Wire CoinRainView into demo app`
- Vor jedem Commit: Tests grün, kein toter Code, keine TODO-Kommentare ohne
  Ticket-Referenz.

---

## Wenn du unsicher bist

- **Architektur-Frage** (z. B. „gehört das in core oder coinrain?"): zurück zur
  Modulstruktur-Tabelle oben.
- **Physik-Frage** (z. B. „warum hüpfen die Münzen?"): zurück zu „Bewusste
  Designentscheidungen". Wahrscheinlich fehlt Sleep-State, Restitution-Slop,
  oder fixer Timestep.
- **Etwas widerspricht dem Prompt:** Frag nach, ändere nicht eigenmächtig.

Lieber einmal zu viel den Prompt nochmal lesen als einen halben Tag in eine
Sackgasse rennen.

## Workflow für nicht-triviale Aufgaben

1. Bei jeder Aufgabe, die mehr als eine Datei betrifft oder unklare Schritte hat, erstelle zuerst `docs/PLAN.md` mit einer Zusammenfassung deines Verständnisses der Aufgabe, den Annahmen und dem geplanten Vorgehen. Stoppe und warte auf meine Bestätigung.

2. Nach meiner Bestätigung erstelle `docs/TASKS.md` mit konkreten Unteraufgaben
   als Markdown-Checkboxes: `- [ ] Task-Beschreibung`

3. Arbeite die Tasks einzeln ab. Nach jedem abgeschlossenen Task:
   - Aktualisiere `TASKS.md` und ändere `- [ ]` zu `- [x]`
   - Füge bei Bedarf eine kurze Notiz hinzu, was gemacht wurde
   - Fahre erst dann mit dem nächsten Task fort

4. Wenn alle Tasks abgehakt sind, gib eine Abschluss-Zusammenfassung.

## Für Bugfixes

Bei Bugfix-Anfragen beginne IMMER mit einer Analyse-Phase in PLAN.md
(Hypothesen, geplante Messungen) und warte auf Bestätigung, bevor du
Code änderst.

---
> Source: [aidev311936/RollingCoin](https://github.com/aidev311936/RollingCoin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
