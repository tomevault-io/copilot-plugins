## essence

> Stundenerfassung: Arbeitszeit-Tracking und Mitarbeiter-Auswertungen

---
description: 
globs: 
alwaysApply: true
---
---
alias: "@stundenerfassung"
description: "Stundenerfassung: Arbeitszeit-Tracking und Mitarbeiter-Auswertungen"
globs:
  - "app/(protected)/stunden/**/*"
  - "app/components/stunden/**/*"
  - "app/lib/api/staff.ts"
---

# Modul: Stundenerfassung

## Zweck
Die Stundenerfassung dient dazu, geplante und tatsächliche Arbeitszeiten der Mitarbeiter zu dokumentieren, zu vergleichen und auszuwerten.

---

## Tabelle: `public.staff_hours`

| Spalte        | Typ         | Beschreibung                                 |
|---------------|-------------|----------------------------------------------|
| id            | UUID        | Primärschlüssel                              |
| staff_id      | UUID        | FK → staff                                   |
| date          | DATE        | Datum der Arbeitszeit                        |
| hours         | DECIMAL(5,2)| Gearbeitete Stunden                          |
| reason        | TEXT        | optional, z. B. "BGM-Termin", "Urlaub", etc. |
| vacation_days | INTEGER     | optionale Urlaubtage                         |
| sick_days     | INTEGER     | optionale Krankheitstage                     |
| created_at    | TIMESTAMP   | automatisch generiert                        |

---

## Funktionen

- Erfassung erfolgt **pro Kalendertag**
- Visualisierung:
  - Wochenansicht: Montag–Sonntag
  - Monatsansicht: Summe pro Tag/Mitarbeiter
- Rollenbasierte Bearbeitung:
  - Mitarbeiter: nur eigene Einträge
  - Studioleiter/Admin: alle Einträge
- Ampelfarben je nach Erfüllung:
  - **Rot:** < 75% der Sollstunden
  - **Gelb:** 75–100%
  - **Grün:** ≥ 100%

---

## Validierung

- Nur **ein Eintrag pro Tag und Mitarbeiter**
- Stundenwerte: `0 ≤ hours ≤ 24`
- Vergangene Monate: Bearbeitung nur durch Admin
- Gesamtstunden + Abwesenheit darf 31 Tage / Monat nicht überschreiten

---

## Erweiterung

- Pro Mitarbeiter und Monat können **Sollstunden** hinterlegt werden
- Vergleich Ist/Soll → farbige Indikatoranzeige
- Aggregation auf Monatsbasis für:
  - Gearbeitete Stunden
  - Urlaubstage
  - Krankheitstage

---

## UI-Komponenten

- `StundenModal`: zur Eingabe & Bearbeitung einzelner Einträge
- `StundenKarte`: KPI-Komponente (z. B. 24/40h + Fortschrittsbalken)
- `StundenTabelle`: Wochen- und Monatsübersicht mit Filter

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xife12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
