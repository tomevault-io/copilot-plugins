## beehive

> - Kommunikation mit User: **Deutsch**

# Beehive — Projektnotizen für Claude

## ⚠️ PERMANENTE REGELN — NIEMALS LÖSCHEN! ⚠️

### Kommunikation
- Kommunikation mit User: **Deutsch**
- Code-Kommentare: Englisch
- App-Sprache: Englisch (primär), deutsche Übersetzung auf Wunsch

### Plattform
- **Primär: macOS** (Android und weitere später)

### Performance (OBERSTE PRIORITÄT!)
- Tausende Bienen ohne Lag — das ist das Ziel
- **Flat Arrays** statt Pointer-Arrays (Cache-freundlich)
- NIEMALS einen Node/Struct per malloc pro Biene/Zelle
- Einfache Lösungen bevorzugen

### ⚠️ DEUTSCHE IDENTIFIER — PFLICHT!
**Alle Bezeichner (Funktionen, Typen, Variablen, Konstanten) auf Deutsch.**
Umlaute ersetzen: ä→ae, ö→oe, ü→ue
Beispiel: `bienenstock_zeichnen()`, `Spielzustand`, `ZELLE_HONIG`, `WiesenBiene`

### Session-Start (ZWINGEND!)
1. `memory/beehive-discipline.md` lesen
2. `memory/beehive-architecture.md` lesen
3. Alle `.h` Dateien lesen (Glob: `src/**/*.h`) — gibt vollständigen Überblick
4. Erst DANN mit User besprechen was heute gemacht wird

### Session-Ende (ZWINGEND!)
- `memory/beehive-architecture.md` updaten (neuer Stand, neue Dateien)
- `memory/MEMORY.md` aktuellen Stand + nächste Schritte updaten

---

## Projektstruktur (aktuell)

```
src/
  main.c                  ← Einstiegspunkt, Spielschleife
  kern/
    spielzustand.h        ← Spielzustand, Ansicht-Enum, Fensterkonstanten
  szenen/
    bienenstock.h/.c      ← Querschnitt-Ansicht (Wabenraster, Bienen im Stock)
    wiese.h/.c            ← Vogelperspektive (Blumen, Flugbienen, Aussenbereich)
assets/                   ← Texturen, Sounds, Fonts (noch leer)
Makefile, CLAUDE.md
```

## Build & Run

```bash
make run       # kompilieren + starten
make           # nur kompilieren
make clean     # aufräumen
```

## Ansichten (TAB zum Wechseln)
- **ANSICHT_STOCK**: Querschnitt des Bienenstocks, Honigwaben, Bienen, Königin
- **ANSICHT_WIESE**: Vogelperspektive, Blumen, fliegende Bienen, Stock-Gebäude

## Geplante Milestones
1. ✅ Visuelles Grundgerüst (Waben, zwei Ansichten)
2. Pheromon-System (Bienen hinterlassen Spuren)
3. Task-System (Sammlerin, Wächterin, Amme, Bauerin)
4. Lebenszyklus (Ei → Larve → Puppe → Biene)
5. Jahreszeiten (Winter-Überlebens-Mechanik)
6. Feinde (Wespen, Hornissen)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Taubenkralle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
