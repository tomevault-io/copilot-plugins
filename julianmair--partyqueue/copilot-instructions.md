## partyqueue

> Diese Datei beschreibt, wie KI-Agenten oder Codex in diesem Projekt arbeiten sollen.


## 2. `AGENTS.md`

```md
# AGENTS.md

Diese Datei beschreibt, wie KI-Agenten oder Codex in diesem Projekt arbeiten sollen.

## Projektziel

PartyQueue ist eine Spotify-basierte Party-Voting-App.

Ein Host verwaltet eine Party und Gäste voten mobil über einen QR-Code.  
Die App soll stabil, einfach erweiterbar und providerfähig aufgebaut bleiben.

## Architekturregeln

### 1. Die interne PartyQueue ist die Quelle der Wahrheit
- Die PartyQueue im `PartyManager` bestimmt die Reihenfolge.
- Spotify dient nur als Abspielsystem.
- Die Spotify-Queue soll nicht als Hauptqueue verwendet werden.

### 2. Voting verändert nur die interne Queue
- Votes sortieren nur die interne PartyQueue.
- Mobile Clients dürfen Songs nur voten, nicht steuern oder abspielen.
- Pro Client darf ein Song nur einmal gevotet werden.

### 3. Mobile Voting ist bewusst reduziert
- Mobile Voting zeigt nur die Top 10 Songs.
- Keine Host-Funktionen auf der Mobile-Seite.
- Keine Spotify-Steuerung auf mobilen Geräten.

### 4. Provider-Architektur erhalten
- Es existiert ein Provider-Pattern, damit später neben Spotify auch andere Anbieter wie Tidal möglich sind.
- Änderungen an Provider-Typen und Factorys vorsichtig durchführen.
- Bestehende Track-Typen wiederverwenden, keine unnötigen parallelen Modelle einführen.

## Wichtige Dateien

- `src/app/lib/party/PartyManager.ts`
  - Queue-Logik
  - Voting-Logik
  - Spotify-Synchronisation
  - Party-State

- `src/app/lib/providers/types.ts`
  - Zentrale Typen wie `Track`, `PartyTrack`, `MusicProvider`

- `src/app/lib/providers/factory.ts`
  - Provider Factory

- `src/app/lib/providers/spotify/`
  - Spotify-spezifische Implementierung

- `src/app/api/auth/login/route.ts`
  - Spotify Login Redirect

- `src/app/api/auth/callback/route.ts`
  - Spotify Callback und Cookie-Handling

- `src/app/api/auth/token/route.ts`
  - Token Refresh

- `src/app/party/[id]/vote/page.tsx`
  - Mobile Voting-Seite

## Wichtige Arbeitsregeln

### Bei Änderungen an Auth
- Redirect URIs exakt prüfen
- Produktionsdomain und HTTPS berücksichtigen
- Docker-/Build-abhängige ENV-Probleme beachten
- Nicht versehentlich lokale URLs hart codieren

### Bei Änderungen an Queue/Playback
- Keine Spotify-Queue als führende Logik einführen
- Die interne PartyQueue bleibt führend
- Doppeltes Einreihen und Race Conditions vermeiden
- Gespielte Songs sauber aus der PartyQueue entfernen

### Bei Änderungen am Frontend
- Mobile-Seite nur für Voting
- Host-Seite darf volle Kontrolle haben
- UI möglichst konsistent zwischen Host und Mobile halten

### Bei Änderungen an Typen
- `Track` und `PartyTrack` konsistent halten
- Keine unnötigen `any`-Typen einführen, wenn sich einfache Typen definieren lassen
- Wenn externe API-Strukturen unklar sind, lieber gezielte Mapping-Typen verwenden

## Deployment-Regeln

- Das Projekt wird per Docker gebaut
- Deployment-Ziel ist Unraid
- Docker-Images werden über GitHub Actions gebaut und zu Docker Hub gepusht
- Produktions-ENV müssen im Build-Kontext berücksichtigt werden
- Spotify OAuth in Produktion braucht HTTPS

## Bekannte Besonderheiten

- Next.js 15 verwendet Promise-basierte `params`
- Mobile Voting nutzt lokale Client-IDs zur Vote-Begrenzung
- Browser können bei frischen Domains oder lokalen HTTPS-Setups Warnungen zeigen
- Spotify Redirect URIs müssen exakt zur Domain und zum Protokoll passen

## Was Agenten vermeiden sollen

- Keine unkontrollierten Refactors der Party- oder Provider-Architektur
- Keine Einführung einer zweiten konkurrierenden Queue-Logik
- Keine Host-Funktionen auf der Mobile-Seite
- Keine Änderungen an Auth-Flows ohne Prüfung der Redirect- und Cookie-Logik

## Aktuelle Prioritäten
- FooterPlayer lädt nicht immer zuverlässig und muss stabilisiert werden
- Lautstärkeregelung im Player soll ergänzt werden
- Party-Zustand darf nach Reload nicht verloren gehen
- PartyQueues sollen künftig verwaltbar und persistent sein
- Es soll möglich werden, ganze Playlists in die PartyQueue zu übernehmen

## Wichtige Produktregeln
- Aktuell können einzelne Songs zur PartyQueue hinzugefügt werden
- Zukünftig soll auch das Hinzufügen ganzer Playlists möglich sein
- PartyQueues sollen nicht nur temporär im RAM existieren
- Nach Reload oder erneutem Öffnen soll eine Party wiederhergestellt werden können
- Nutzer sollen vorhandene PartyQueues verwalten können

---
> Source: [JulianMair/PartyQueue](https://github.com/JulianMair/PartyQueue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
