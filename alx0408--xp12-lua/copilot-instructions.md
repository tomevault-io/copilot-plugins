## xp12-lua

> - Skriptsprache: Lua (FlyWithLua NG+ für X-Plane 12)

# CLAUDE.md – Projektregeln XP12-LUA

## Umgebung
- Skriptsprache: Lua (FlyWithLua NG+ für X-Plane 12)
- Entwicklung auf macOS, Ausführung auf Windows (X-Plane 12)

## FlyWithLua: Globale vs. lokale Funktionen

Funktionen die via String registriert werden, müssen **global** sein (kein `local`).
FlyWithLua wertet diese Strings im globalen Scope aus — lokale Funktionen sind dort unsichtbar.

Betrifft alle Registrierungsfunktionen:
- `do_sometimes("func()")`
- `do_often("func()")`
- `do_every_draw("func()")`
- `create_command(..., "func()", ...)`
- `add_macro(..., "code", "code", ...)`

**Richtig:**
```lua
function pitot_tick() ... end
do_sometimes("pitot_tick()")
```

**Falsch:**
```lua
local function pitot_tick() ... end  -- unsichtbar für FlyWithLua
do_sometimes("pitot_tick()")
```

Hilfsfunktionen die nur direkt aufgerufen werden, dürfen `local` bleiben.

## FlyWithLua: Gültige Callback-Funktionen

Nur diese Registrierungsfunktionen existieren in FlyWithLua NG+:

| Funktion | Aufrufhäufigkeit |
|---|---|
| `do_every_draw(s)` | Jeden Frame |
| `do_often(s)` | ~10× pro Sekunde |
| `do_sometimes(s)` | ~1× pro Sekunde |
| `do_on_keystroke(s)` | Bei Tastendruck |
| `do_on_mouse_click(s)` | Bei Mausklick |
| `create_command(name, desc, begin, cont, end)` | Command-Handler |
| `add_macro(name, on, off, default)` | Makro-Eintrag |

**Nicht verwenden** (existiert nicht):
- ~~`do_on_airport_load()`~~
- ~~`do_on_new_flight()`~~
| ~~`do_rarely(s)`~~ | existiert nicht — stattdessen `do_sometimes` mit manuellem Throttle |

## Projekt-Philosophie
- DataRefs nur zurücksetzen wenn sie auf den erwarteten Fehlerwert gesetzt sind
- Schrittweise vorgehen, ein Modul nach dem anderen

## Arbeitsweise (Tools & Nachfragen)
Gilt für dieses gesamte Repo, in jeder Session.

- **Dateien lesen/durchsuchen:** Read/Grep/Glob-Tools verwenden, NICHT `bash cat/grep/sed`. Die Dateien liegen im Repo — direkt darauf arbeiten (das fragt nicht nach).
- **Bash nicht mit `cd "…" &&` einleiten.** Das Arbeitsverzeichnis ist bereits gesetzt; `cd` in einem zusammengesetzten Befehl löst unnötige Berechtigungs-Prompts aus. Direkt aufrufen oder absolute Pfade nutzen.
- **Dateien ändern (auch kleine `.prf`) mit dem Edit-Tool** — genau wie bei Lua. Kein Python fürs Auslesen oder für kleine Änderungen.
- **Python nur für seltene Massen-Umbauten** (viele Buttons gleichzeitig, Über-Kreuz-Tausch-Logik). Diese Python-Aufrufe werden **einzeln bestätigt** — kein pauschales `Bash(python3 *)` in der Allowlist.
- **Nachfragen:** Bei echter Unklarheit **immer** kurz fragen (wichtig!). Sonst: entscheiden → umsetzen → knapp berichten und getroffene Annahmen im Bericht nennen. Nicht wegen jeder Kleinigkeit stoppen.
- **Commit/Push nur auf ausdrückliche Anweisung.**

---
> Source: [alx0408/XP12-LUA](https://github.com/alx0408/XP12-LUA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
