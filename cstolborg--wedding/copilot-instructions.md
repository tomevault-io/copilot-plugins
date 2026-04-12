## wedding

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Projektoversigt

Statisk bryllupshjemmeside for Sara Amalie & Christians bryllup d. 6. juni 2026 på Sødringholm i Havndal, Danmark. Siden er skrevet på dansk.

## Arkitektur

Hele hjemmesiden er en enkelt `index.html` fil med:
- Inline CSS styles i `<head>` sektionen
- Semantiske HTML sektioner: hero, detaljer, program, aktiviteter, overnatning og RSVP
- Responsivt design med mobile breakpoints ved 768px
- Farveskema baseret på salvie-grøn (#6b7a65) og varme neutraler

## Udvikling

Ingen build-værktøjer eller dependencies. Åbn `index.html` direkte i en browser for at se siden.

## Nøglesektioner

- **Hero**: Navne, dato, lokation
- **Bryllupsdetaljer**: Lokation (med Google Maps link), dress code
- **Program**: Opdelt i tre dage:
  - Fredag (valgfri): Ankomst fra 16:00, fredagsreception 18:00-24:00
  - Lørdag (bryllupsdagen): Vielse 14:00, reception 16:00, middag 18:00, fest 23:00
  - Søndag: Morgenmad 10:00, afrejse 12:00
- **Aktiviteter**: ATV-kørsel, tennisturnering, lerdueskydning
- **Overnatning**: På gården eller Hotel Randers
- **RSVP**: Link til Google Forms (deadline 1. maj 2026)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cstolborg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Cstolborg)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
