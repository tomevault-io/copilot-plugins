## nvl-core

> Dieses Repository ist **NuVio Core**.

# CLAUDE.md

## Projekt

Dieses Repository ist **NuVio Core**.

NuVio Core ist der gemeinsame, branchenneutrale Betriebskern der NuVio Plattform.  
Es ist keine Einzellösung für einen Kunden, sondern ein skalierbares, professionell wartbares **Multi Tenant SaaS Fundament**, auf dem später spezialisierte Produkte wie **NuVio Fleet** aufsetzen.

Ziel ist nicht nur eine funktionierende Admin Oberfläche, sondern ein strukturell sauberes, agenturfähiges und produktisierbares System.

---

## Kernziel

Jede Änderung in diesem Repository soll NuVio Core näher an dieses Ziel bringen:

- mandantenfähig ab Fundament
- datengetrieben statt hart codiert
- professionell wartbar
- modular erweiterbar
- ohne harte Sonderlogik für einzelne Kunden
- geeignet für produktive Kundenumgebungen
- geeignet für spätere vertikale Produkte auf gemeinsamer Core Basis

Wichtig:
Nicht nur funktional richtig bauen.  
Sondern **strukturell richtig und dsgvo konform und sicher**.

---

## Technischer Kontext

Aktueller Stack:

- React
- TypeScript
- Vite
- React Router
- Supabase
- TanStack React Query
- React Hook Form
- Zod
- Tailwind CSS
- Radix UI
- Lucide
- Framer Motion
- Vitest

Projektstruktur grob:

- `src/app` für App Aufbau, Router, Navigation, Provider
- `src/features` für fachliche Module
- `src/lib` für Supabase, Query Client, Utilities und generierte Typen
- `src/pages` für Seitenebene
- `supabase/sql` für SQL und Datenbanklogik

Wichtige fachliche Module aktuell:

- auth
- bookings
- pricing
- resources
- workspace

---

## Produktgrenzen von NuVio Core

NuVio Core enthält den allgemeinen Betriebskern, insbesondere:

- Authentifizierung und Session Handling
- Workspace und aktiver Mandantenkontext
- Benutzer, Memberships, Rollen und Berechtigungen
- Kundenverwaltung
- Ressourcenverwaltung
- Preislisten und Preislogik
- Buchungen und Kalenderlogik
- Standorte und Übergabepunkte
- Vertrags und Dokumentenbasis
- Uploads und Dateiverwaltung
- OCR und Dokumentenerkennung mit Bestätigungsworkflow
- Abrechnungsgrundlagen
- auditierbare Änderungslogik
- Einstellungen pro Mandant
- optionale White Label und Agenturkonfiguration

Nicht in den Core gehören:

- harte Sonderlogik für einzelne Kunden
- branchenspezifische Frontend Sonderpfade
- dauerhaft verstreute Preislogik im Frontend
- UI Workarounds statt sauberer Modellierung
- doppelte Wahrheiten bei Types, Queries oder Zuständen
- neue Patterns ohne klaren architektonischen Nutzen

Spätere Produkte wie Fleet bauen auf Core auf und ergänzen nur branchenspezifische Fachlogik.

---

## Architekturregeln

### 1. Multi Tenant ist nicht optional
Mandantenfähigkeit ist Kern des Systems.

Die zentrale Wahrheit läuft über:

- `company_id`
- `slug`
- `memberships`
- `activeCompany`
- `activeRole`

Jeder fachliche Datensatz, jede Sichtbarkeit und jede Berechtigung muss sauber mandantenfähig gedacht werden.

### 2. Workspace ist zentral
Workspace ist die Einstiegsschicht für aktiven Mandanten, aktive Membership und aktive Rolle.

Zentrale Zustände:

- `activeCompanyId`
- `activeCompany`
- `activeMembership`
- `activeRole`

Änderungen an Routing, Guards, Filtern oder Datenzugriffen dürfen diese Logik nicht umgehen.

### 3. Keine harte Sonderlogik pro Kunde
Unterschiede zwischen Kunden sollen nicht über Frontend Sonderpfade gelöst werden.

Bevorzugte Mittel:

- Konfiguration
- Datenmodell
- Module
- Rollen
- Preislisten
- Mandanteneinstellungen

### 4. Klare Trennung der Schichten
Trenne sauber zwischen:

- Domain
- Datenzugriff
- Applikationslogik
- UI

UI Komponenten sollen präsentationsnah bleiben.  
Business Logik gehört in Services, Mapper, Validierungen und klar benannte fachliche Schichten.

### 5. Bestehende Muster respektieren
Erweitere bestehende:

- Services
- Hooks
- Types
- Mapper
- Queries

Erzeuge keine Parallelstrukturen, wenn die bestehende Struktur fachlich sinnvoll erweitert werden kann.

### 6. Kleine, sichere Änderungen vor großen Umbauten
Bevorzuge minimale, nachvollziehbare Änderungen.

Keine unnötigen Refactors.  
Keine neuen Abstraktionen ohne klaren Mehrwert.  
Keine spekulativen Umbauten.

---

## Wichtige Invarianten

Diese Bereiche gelten als besonders sensibel und dürfen nicht leichtfertig verändert werden:

- Workspace Zuordnung
- Membership Ermittlung
- `profile_id` basierte Membership Logik
- `company_id` basierte Filterung
- aktive Rollenlogik
- fachlich korrekte Mandantentrennung
- Booking Kernlogik

Wenn du in diesen Bereichen arbeitest:

- prüfe zuerst bestehende Typen, Services und Queries
- ändere so wenig wie möglich
- vermeide Seiteneffekte auf andere Module
- halte Änderungen testbar und nachvollziehbar

---

## Booking Modul Hinweise

Das Booking Modul ist ein besonders wichtiger Bereich.

Dort gilt:

- risikoarm arbeiten
- bestehende Logik nicht breit umbauen
- Validierung, Konfliktlogik und Custom Field Sync nicht unbedacht verändern
- vorhandene Tests respektieren
- bei Änderungen gezielt nur den kleinsten sinnvollen Bereich anfassen

---

## Datenzugriff und Supabase

Beim Arbeiten mit Supabase gilt:

- Queries fachlich sauber und mandantenfähig aufbauen
- `company_id` als zentrale Zuordnung respektieren
- keine Datenzugriffe ohne klaren Workspace Bezug
- generierte Typen, App Types und Domain Types logisch konsistent halten
- keine doppelten Datenmodelle einführen
- Mapper nur dort ergänzen, wo sie wirklich fachlich nötig sind

Wenn möglich:

- bestehende Service Dateien erweitern
- bestehende Query Keys und Query Flows respektieren
- neue Abfragen nur dann einführen, wenn die vorhandene Struktur nicht ausreicht

---

## Arbeitsweise für Änderungen

Bei jeder Aufgabe:

1. zuerst bestehende Struktur prüfen
2. kleinsten sinnvollen Eingriff identifizieren
3. bestehende Dateien bevorzugen
4. nur gezielt neue Dateien anlegen, wenn es fachlich nötig ist
5. Typen, Services und UI konsistent halten
6. Risiko für Mandantenlogik, Rollenlogik und Buchungslogik mitdenken

Bevorzugt:

- kleine Folgeänderungen
- klare Benennung
- geringe Diff Größe
- nachvollziehbare Datenflüsse
- wartbare Lösungen

Vermeiden:

- breite Refactors
- neue Pattern ohne Grund
- versteckte Logik in UI Komponenten
- schnelle Workarounds mit späteren Folgekosten

---

## Qualitätsmaßstab

Jede Änderung soll:

- mandantenfähig sein
- wartbar sein
- fachlich nachvollziehbar sein
- testfreundlich sein
- konsistente Types verwenden
- bestehende Architektur stärken
- keine doppelte Wahrheit erzeugen

Refactors nur dann, wenn sie:

- echten architektonischen Mehrwert liefern
- klar abgegrenzt sind
- geringes Risiko haben
- bestehende Flows nicht destabilisieren

---

## Erwartete Ausgabe bei Umsetzungsarbeit

Wenn du eine Aufgabe bearbeitest, arbeite konkret, fokussiert und ohne generische Ausschweifungen.

Bevorzugtes Vorgehen:

- kleinsten sinnvollen Schritt umsetzen
- nur relevante Dateien anfassen
- bestehende Struktur respektieren
- Änderungen kurz fachlich begründen
- Risiken oder Prüfstellen knapp benennen

Wenn ein Vorhaben zu groß ist, teile es in kleine logisch geordnete Schritte auf, statt einen großen Umbau vorzuschlagen.

---

## Was du nicht tun sollst

- keine Sonderlogik für einzelne Kunden fest einbauen
- keine Mandantenlogik umgehen
- keine Preislogik dauerhaft im Frontend verstreuen
- keine konkurrierenden Zustände einführen
- keine unnötigen neuen Dateien anlegen
- keine neuen Architekturpattern ohne klaren Nutzen einführen
- keine UI Logik mit versteckter Business Logik vermischen
- keine spekulativen Änderungen außerhalb der eigentlichen Aufgabe vornehmen

---

## Entscheidungsregel bei Unsicherheit

Wenn mehrere Wege möglich sind, wähle den Weg, der:

- die bestehende Struktur am wenigsten bricht
- mandantenfähig sauber bleibt
- mit vorhandenen Types, Services und Queries harmoniert
- die geringste langfristige Komplexität erzeugt
- den kleinsten sicheren Schritt darstellt

---

## Kurzform

Baue NuVio Core wie ein hochwertiges, skalierbares Multi Tenant SaaS Fundament.

Nicht kundenindividuell improvisiert.  
Nicht schnell zusammengebaut.  
Nicht nur funktional.  

Sondern:

- strukturell sauber
- datengetrieben
- modular
- wartbar
- professionell erweiterbar

---
> Source: [NuVio-Labs/NVL_Core](https://github.com/NuVio-Labs/NVL_Core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
