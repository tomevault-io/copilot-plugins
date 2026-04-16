## fltr-dashboard

> Hallo! Hier ist der Plan für die Entwicklung deines React-Dashboards zur Verwaltung von Reality-TV-Daten.

# Gemini Reality-TV Dashboard Plan

Hallo! Hier ist der Plan für die Entwicklung deines React-Dashboards zur Verwaltung von Reality-TV-Daten.

## Planungsphase

### 1. Technologie-Stack und Setup
- **Framework:** React.js (mit Vite für eine schnelle Entwicklungsumgebung).
- **UI-Bibliothek:** Ant Design (`antd`) für professionelle und schnell einsetzbare Dashboard-Komponenten (Tabellen, Formulare, Modals, etc.).
- **Routing:** `react-router-dom` für die Navigation zwischen den verschiedenen Seiten des Dashboards.
- **Styling:** Standard-Styling von Ant Design, eventuell mit kleinen Anpassungen.

### 2. Projektstruktur
Ich werde eine klare und skalierbare Ordnerstruktur anlegen:
```
/src
|-- /api (später für die API-Anbindung)
|-- /assets
|-- /components (Wiederverwendbare UI-Elemente, z.B. AddPersonModal)
|-- /data (Dummy-Daten für Shows, Staffeln, Personen)
|-- /layouts (Haupt-Layout für das Dashboard, z.B. mit Sidebar)
|-- /pages (Die Hauptansichten: Shows, Personen, etc.)
|-- /routes (Konfiguration des Routings)
|-- App.jsx
|-- main.jsx
```

### 3. Dummy-Daten
Ich erstelle Beispieldaten, die die Struktur deiner API (`/shows`, `/seasons`, `/persons`) widerspiegeln.
- `shows.js`: Eine Liste von Shows (z.B. "Ich bin ein Star – Holt mich hier raus!", "Das Sommerhaus der Stars").
- `seasons.js`: Eine Liste von Staffeln, jeweils mit einer `showId`.
- `persons.js`: Eine Liste von Teilnehmern, jeweils mit einer `seasonId`.

### 4. Implementierungs-Schritte

1.  **Grund-Setup:** Ein neues React-Projekt mit Vite aufsetzen und die Abhängigkeiten (`antd`, `react-router-dom`) installieren.
2.  **Layout & Navigation:** Ein Haupt-Layout mit einer persistenten Sidebar erstellen. Die Sidebar wird Links zu den Hauptbereichen enthalten:
    - Dashboard (Übersicht)
    - Shows
    - Personen
    - Suche/Discover
3.  **Seiten-Implementierung (mit Dummy-Daten):**
    - **Shows-Seite:** Eine Tabellenansicht aller Shows. Ein Klick auf eine Show führt zur Detailseite. Ein "Show hinzufügen" Button öffnet ein Modal.
    - **Show-Detailseite:** Zeigt Details einer Show und eine Liste ihrer Staffeln. Hier gibt es einen "Staffel hinzufügen" Button.
    - **Staffel-Detailseite:** Zeigt Details einer Staffel und eine Liste der Teilnehmer. Hier gibt es einen "Teilnehmer hinzufügen" Button.
    - **Personen-Seite:** Eine umfassende Liste aller Teilnehmer aus allen Shows. Diese Seite wird die Kernfunktionalität zum Verwalten und Zusammenführen von Personen enthalten.
4.  **CRUD-Funktionalität (UI):**
    - Implementierung von Modals und Formularen (mit `antd`) zum Hinzufügen und Bearbeiten von Shows, Staffeln und Teilnehmern. Die Aktionen werden vorerst nur die Dummy-Daten im Frontend-State manipulieren.
5.  **Personen zusammenführen (Merging):**
    - Auf der "Personen"-Seite wird es eine Funktion geben, um zwei oder mehr Teilnehmer auszuwählen.
    - Nach der Auswahl öffnet sich ein Modal, in dem du eine "Master"-Person auswählen kannst und die Daten der anderen zusammengeführt werden.
    - Diese Funktion wird zunächst als reiner UI-Prototyp umgesetzt.
6.  **Suchen/Discover:**
    - Eine einfache Suchseite mit einem Eingabefeld, die die Dummy-Daten nach Shows, Staffeln oder Personen durchsucht und die Ergebnisse anzeigt.

---

## Klärende Fragen

Bevor ich mit der Implementierung beginne, habe ich drei kurze Fragen, um sicherzustellen, dass ich deine Vorstellungen genau treffe:

1.  **Zum Zusammenführen von Personen:** Wie stellst du dir den Prozess genau vor? Soll ich zwei Personen aus einer Liste auswählen können, und dann in einem Dialogfeld entscheiden, welche Daten (z.B. Name, Geburtsdatum) vom einen oder anderen Datensatz übernommen werden? Oder reicht vorerst eine einfachere Logik (z.B. Person A wird gelöscht und alle ihre Verknüpfungen gehen an Person B)?
ja richtig

2.  **Zur "Discover"-Suche:** Was genau soll durchsucht werden? Nur die Namen der Personen? Oder auch Shows und Staffeln? Sollen alle Ergebnisse gemischt angezeigt werden?
die suche läuft übers backend mit /discover. Diese ist schon implementiert. Mach da eine Placeholder URL rein. Das ergebnis sieht so aus:
{
        "type": "SHOW",
        "data": {
            "showId": 1,
            "title": "Are You The One?",
            "genre": "Reality, Dating",
            "bannerUrl": "https://streamcoimg-a.akamaihd.net/000/123/147/123147-Banner-L2-b81d3e6b4df34af2887f701eec382f87.jpg",
            "thumbnailUrl": "https://www.kinoundco.de/image/posters/6lapdrxzuECSAF37PUHDnrsMm9x.jpg",
            "logoUrl": "https://upload.wikimedia.org/wikipedia/commons/0/03/Are-you-the-one.png",
            "description": "Je zehn Single-Frauen und -Männer treffen im traumhaften Südafrika auf ihr perfektes Match. Die Teilnehmer müssen selbst herausfinden, wer zu wem gehört. Nur wenn alle zusammen richtig liegen, gewinnen sie eine Geldsumme in Höhe von 200.000 Euro.",
            "concept": "Die Teilnehmer gehen auf Dates, die durch Wettbewerbe bestimmt werden, und haben die Möglichkeit, im \"Wahrheitsraum\" zu erfahren, ob ein bestimmtes Paar ein \"Perfect Match\" ist. Bestätigte Paare ziehen in die \"Honeymoon-Suite\" und sind für den Rest der Staffel automatisch zusammen. Am Ende jeder Episode gibt es eine \"Matching Night\", in der die Teilnehmer erfahren, wie viele perfekte Paare sie gefunden haben, jedoch nicht, welche Paare korrekt sind.",
            "streamingServices": "RTL+",
            "startDate": "2020-04-14",
            "endDate": null
        }
    },

3.  **Zum Design:** Hast du spezielle Wünsche für das Design und die Anmutung des Dashboards, oder soll ich ein sauberes, modernes und standardmäßiges Dashboard-Design mit Ant Design umsetzen?
standard

Sobald ich deine Antworten habe, kann ich mit dem Aufsetzen des Projekts beginnen.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mafeth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
