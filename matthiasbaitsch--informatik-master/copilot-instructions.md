## informatik-master

> Dieses Projekt enthält die Lehrveranstaltungsunterlagen für das Modul **Informatik im Master Bauingenieurwesen** an der **Hochschule Bochum**.

# Vorlesungsunterlagen Informatik – Master Bauingenieurwesen

## Kontext

Dieses Projekt enthält die Lehrveranstaltungsunterlagen für das Modul **Informatik im Master Bauingenieurwesen** an der **Hochschule Bochum**.

Die Studierenden lernen **objektorientiertes Programmieren mit C#**. Zielgruppe sind Bauingenieur-Masterstudierende ohne vertiefte Programmiererfahrung.

## Lernziele

- Studierende sollen die vermittelten Konzepte **selbständig anwenden** können – Verständnis und Transfer stehen im Vordergrund, nicht das Auswendiglernen von Syntax
- Das Modul legt eine **solide konzeptuelle Basis**; Sprachdetails können Studierende bei Bedarf später selbständig erarbeiten
- Der behandelte Sprachumfang beschränkt sich auf die **wesentlichen OOP-Konzepte**: Klassen, Objekte, Kapselung, Vererbung, Polymorphismus
- Komplexe oder randständige C#-Features werden bewusst weggelassen

## Projektstruktur

- `lernpfad/` ist automatisch generiert – nie manuell bearbeiten. Alle Änderungen gehen in `bausteine/`.
- Skripte (Julia) liegen in `skripte/` und dürfen direkt bearbeitet werden.
- Manche Dateien existieren bewusst mehrfach (z.B. in Startcode und Musterlösung). `skripte/duplikate.txt` listet die inhaltsgleichen Gruppen; der Pre-Commit-Hook `githooks/pre-commit` erzeugt die Liste mit `julia skripte/duplikate.jl` neu und schlägt an, wenn sie sich dabei ändert – also wenn eine Kopie einseitig geändert wurde oder Duplikate hinzugekommen/weggefallen sind.

## Technischer Aufbau

- **Format:** Quarto (`.qmd`), Deutsch
- **Ausgabeformate:** Reveal.js-Folien und HTML-Seiten (konfiguriert in `_quarto.yml`)
- **Struktur pro Themenpaket:** Verzeichnis `XX-thema/` mit
  - `about.yml` – Lernziele und Übersicht des Pakets
  - `folien/folien.qmd` – Vorlesungsfolien
  - `aufgaben/*.qmd` – Aufgaben und Projektaufgaben
  - `aufgaben/projekt/` – Aufgaben-Startcode
  - `aufgaben/projekt-musterloesung/` – Musterlösung
- **Entwicklungsumgebung der Studierenden:** Visual Studio Code mit C# Dev Kit

## Zusammenarbeit mit dem Nutzer

- Den Nutzer duzen
- „sauber" nicht als Qualitätsurteil verwenden (z.B. nicht „das Dokument ist sauber") – stattdessen „gut", „stimmig", „nichts anzumerken" o.Ä.

## Hinweise zur Zusammenarbeit

- Notizen und Erinnerungen gehören in diese Datei (`CLAUDE.md`)
- Kein Memory-Verzeichnis anlegen – weder im Projekt noch anderswo. Das eingebaute auto-memory-System nicht verwenden.

## Arbeitshinweise für Claude

- Linter-Hinweise in `.qmd`-Dateien ignorieren – das macht der Nutzer selbst
- Reihenfolge Beispiel vor Erklärung nicht als Problem werten – didaktisch bewusst so
- CSS-Abstände in `style-slides.scss` immer über `margin-top` des nachfolgenden Elements steuern, nie über `margin-bottom` – Reveal.js nutzt Flexbox, dort kollabieren Margins nicht und addieren sich sonst unkontrolliert
- In nummerierten Aufgabenlisten keine Fettschrift für Schrittbezeichnungen – also `1. Titel. Text`, nicht `1. **Titel.** Text`
- Als Dezimaltrennzeichen wird durchgehend der Punkt verwendet (also `0.75`, nicht `0,75` – auch in Fließtext und Matheumgebungen)
- Reveal.js-Fragments: Soll Inhalt A beim Öffnen der Folie sichtbar sein und auf einen Klick durch Inhalt B ersetzt werden, beide in ein `r-stack` packen, A mit `.fragment .fade-out fragment-index=1`, B mit `.fragment .fade-in fragment-index=1` – gleicher Index sorgt für simultanen Übergang
- In Musterlösungen Kapselung (private Felder/Properties) nicht als fehlend anmerken – Felder bleiben zu diesem Zeitpunkt bewusst öffentlich, Kapselung bringt den Studierenden hier noch keinen Lerngewinn
- Bei Code-Reviews von Musterlösungen: Anzahl neuer Konzepte begrenzen – Refactorings, die nur DRY/Cleanliness verbessern (z.B. Datenlisten deduplizieren) aber neue Sprachkonstrukte einführen würden, nicht vorschlagen

## Inhaltliche Richtlinien

- Alle Texte auf **Deutsch** (Siezen: „Sie")
- C#-Code soll klar und lesbar sein – keine cleveren Shortcuts, keine fortgeschrittenen Features ohne expliziten Grund
- **Bezeichner** (Variablen, Methoden, Klassen) auf **Englisch**, **Kommentare** auf **Deutsch**
- Ordner- und Dateinamen in C#-Projekten pragmatisch: Einführungspakete (wo Namen Lernkonzepte bezeichnen) auf Deutsch, ab OOP-Paketen englisch
- Aufgaben sollen schrittweise aufgebaut sein und auf ingenieurwissenschaftliche Anwendungsfälle eingehen, wo sinnvoll
- Musterlösungen in `projekt-musterloesung/` sollen dem Kenntnisstand der Studierenden entsprechen

---
> Source: [matthiasbaitsch/informatik-master](https://github.com/matthiasbaitsch/informatik-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
