## olivalle-webshop

> > Gemeinsame Arbeitsregeln (Workflow, Code-Qualität, Dokumentation) sind in `../CLAUDE.md` definiert.

# Olivalle Webshop — Claude Code Kontext

> Gemeinsame Arbeitsregeln (Workflow, Code-Qualität, Dokumentation) sind in `../CLAUDE.md` definiert.

> Für alle Aktionen → `make help` (zeigt alle verfügbaren Befehle)

## Über das Projekt
Webshop für "Olivalle" — Verkauf von biologischem Olivenöl (Import aus Andalusien, Spanien).
Wird für einen Freund (Auftraggeber/Inhaber) gebaut. Einzelunternehmer in der Schweiz.
Ersetzt den bisherigen manuellen Bestellprozess via Tally-Formular.

Private Infos (URLs, Zugangsdaten): siehe `NOTES.local.md` (nicht im Repo)

## Produkte & Preise
| Produkt | Preis |
|---|---|
| 250ml Flasche | CHF 8 |
| 500ml Geschenkflasche | CHF 25 |
| 750ml Flasche | CHF 18 |
| 3l Kanister | CHF 50 |

## Test-Strategie
**pytest** für Unit + Integration Tests. Mindestens abgedeckt: Bestelllogik, Stripe Webhook, API-Endpunkte.

## E-Mail-Dienst: Brevo (ehemals Sendinblue)
| Plan | Preis | Limit |
|---|---|---|
| Free | €0/Mt | 9'000 Mails/Mt, max. 300/Tag |
| Starter | €9/Mt | 5'000+ Mails/Mt (kein Tageslimit) |

Für den Stakeholder: Bei ca. 100 Bestellungen/Mt bleibt man im Free Tier.
Entscheid dokumentiert in: `docs/adr-email-provider.md`

## Tech-Stack
| Layer | Tool | Begründung |
|---|---|---|
| Backend/API | Python + FastAPI | Entwickler kennt Python/SQL |
| Frontend | Jinja2-Templates + Tailwind CSS | Kein zweites Framework, alles Python |
| Datenbank | SQLite | Eine Datei, kein separater Service, reicht für ~100 Bestellungen/Mt |
| Payments | Stripe | Twint (CH), Kreditkarte |
| QR-Rechnung | swiss-qr-bill (Open Source) | Kein Bexio, direkt im Code |
| Styling | Tailwind CSS | Utility-first, flexibel |
| Hosting | fly.io (1 Docker-Container, 24/7 seit #116) | Günstig (~$2/Mt real; ursprünglich ~$5/Mt geschätzt), kommerziell erlaubt |

## Design
- **Font:** Amatic SC (Hausschrift der bestehenden Website)
- **Farben:** Weiss auf dunklem Hintergrund, Akzentfarbe Gelb `#f1d600`
- **Logo:** olivalle-logo2017_2.jpg (rundes Logo)

### Tailwind Card-UI Klassen (Issue #51)
| Element | Klassen |
|---|---|
| Card | `bg-stone-700 rounded-lg p-6 shadow-md` |
| Card Hover (Produktkarten) | `hover:shadow-lg hover:-translate-y-1 transition-all duration-200` |
| Responsive Grid | `grid gap-6 sm:grid-cols-2 lg:grid-cols-4` |
| Button Transition | `transition-colors` |

## Wichtigste Funktionen
1. Webshop mit Warenkorb
2. Direkte Zahlung via Stripe (Twint, Kreditkarte)
3. QR-Rechnung via swiss-qr-bill
4. Automatisierte Rechnungsstellung
5. Administrativen Aufwand für Einzelunternehmer minimieren

## Kundendaten
Pflichtfelder: Vorname, Nachname, Strasse, PLZ, Ort, E-Mail
Optionale Felder: Telefonnummer, Kommentar
Versandoptionen: Abholung in der Region Olten / Postversand (CHF 9.90, gratis ab CHF 100)

## Context-Scopes

Je nach Aufgabe nur den relevanten Scope laden — reduziert Token-Verbrauch und hält den Fokus:

| Scope | Pfade | Wann verwenden |
|---|---|---|
| App | `app/`, `templates/`, `static/`, `CLAUDE.md` | FastAPI, Jinja2, Stripe Webhook |
| Vollständig | alles | Architektur- und Querschnittsthemen |

## Architektur-Regeln
- Alles in FastAPI — kein separates Frontend-Framework
- UI-Texte auf Deutsch (CH)

## Status & Aufgaben
**Live auf [olivalle.ch](https://olivalle.ch) seit 2026-04-08 (v1.0).** Phasen 0–3 abgeschlossen.
Laufende Aufgaben werden via [GitHub Issues](https://github.com/konstantinniedermann/olivalle-webshop/issues) verwaltet (Historie unter Milestones).

## Dokumentation
- Übersicht aller Dokumente: `docs/index.md`
- Architektur: `docs/arc42.md`
- Historische Artefakte: `docs/archiv/` (per `.claudeignore` aus Auto-Context ausgeschlossen, bei Bedarf explizit lesen)

## Git & GitHub
- Repository: https://github.com/konstantinniedermann/olivalle-webshop
- Branch-Strategie: `main` (produktiv) — Feature-Branches via PR

## Wichtige Hinweise
- Schweizer Rechtslage: MWST, Datenschutz (DSG)
- Stripe unterstützt Twint nativ in der Schweiz

---
> Source: [konstantinniedermann/olivalle-webshop](https://github.com/konstantinniedermann/olivalle-webshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
