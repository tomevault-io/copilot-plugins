## twopointo

> - Phase 0 (Specs) ist abgeschlossen: docs/ (19 Dateien), ADR/ (12 Dateien), CLAUDE.md, permissions.json existieren.

# twopointo (2.0) – Codex Agent Instructions

## Projektstatus
- Phase 0 (Specs) ist abgeschlossen: docs/ (19 Dateien), ADR/ (12 Dateien), CLAUDE.md, permissions.json existieren.
- Wichtige Patches wurden bereits eingearbeitet:
  - Slash-Commands sind kanonisch inkl. führendem "/" (keine Transformation).
  - "Three Planes": Control Plane (über Edge), Media Plane (SFU direkt), Asset Plane (Object Storage direkt).
  - Asset Deletion Semantik: Resolver gibt 302 auf presigned URL mit TTL 60s; Token wird sofort revoked; Worst-case Zugriff bis TTL falls URL bereits geleakt.
  - Signed mirrors unterstützen JWKS, Release Manifest, Directory Snapshot, Deny/Revocation mit Tombstones + latest-wins.

## Harte Regeln (nicht diskutieren, nicht ändern ohne ADR)
- Keine Federation-Identitäten: immer offizielle Accounts (wichtigster Kernpunkt).
- Pflicht: 2FA.
- E2EE nur in 1:1 DMs und Group DMs, nicht in Servern. Assets/Links sind NICHT verschlüsselt (Banner Pflicht).
- Self-host später: 7-Tage Updatepflicht (nicht in Phase 1 implementieren).
- Standard-Port: 60420.
- Apps/Bots: App Agent läuft auf eigener Hardware, verbindet outbound zum App Gateway (443). Keine Webhooks, keine inbound Ports. Slash-only.
- permissions.json ist Single Source of Truth (vollständiges Discord Permission Bitset, Generator Pflicht).

## Aktuelles Ziel (JETZT)
- Kein Coding.
- Ein reiner Review-Pass über die existierenden Phase-0 Dokumente:
  - docs/OPEN_QUESTIONS_AND_GAPS.md
  - docs/DECISIONS_NEEDED.md
  - docs/RISK_REGISTER.md

## Arbeitsweise für Review-Pass
- Nutze ausschließlich existierende Dateien im Repo als Source of Truth.
- Keine neuen Features erfinden. Nur Lücken/Unklarheiten/Widersprüche herausarbeiten.
- Inkrementell: Wenn eine Output-Datei existiert, ergänze nur Fehlendes und markiere neue Abschnitte mit [ADD].
- Arbeite in kleinen Schritten: pro Run maximal 1 Datei finalisieren, dann stoppen.
- Wenn Kontext/Usage knapp wird: kürze die Anzahl der Punkte, aber beende die aktuelle Datei sauber.

## Git
- Nichts committen ohne explizite Anweisung des Users.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rotstein007)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Rotstein007)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
