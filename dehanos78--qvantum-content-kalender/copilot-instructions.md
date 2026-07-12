## qvantum-content-kalender

> Een interactief content planning dashboard voor Qvantum's marketing team. Het is een single-file HTML applicatie (`Qvantum_Content_Kalender_v1.html`) die de volledige contentplanning beheert: social media posts, campagnes, deadlines en publicatieschema's.

# Qvantum Content Kalender

## Wat dit project is
Een interactief content planning dashboard voor Qvantum's marketing team. Het is een single-file HTML applicatie (`Qvantum_Content_Kalender_v1.html`) die de volledige contentplanning beheert: social media posts, campagnes, deadlines en publicatieschema's.

## Tech stack
- **Type:** Single-file HTML/CSS/JavaScript applicatie (143 KB)
- **Framework:** Vanilla JS (geen build tools, geen npm)
- **Styling:** Inline CSS, Qvantum brand kleuren
- **Data:** Embedded in HTML of via localStorage
- **Geen server nodig** — opent direct in de browser

## Architectuur
```
qvantum-content-kalender/
├── .claude/                           ← Claude Code configuratie
├── Qvantum_Content_Kalender_v1.html   ← De volledige applicatie
└── .gitignore
```

Dit is bewust een single-file app. Alle logica, styling en data zitten in één HTML bestand. Dit maakt het eenvoudig te delen, te openen en te gebruiken zonder installatie.

## Scope-grenzen — KRITISCH
- Werk UITSLUITEND in `dashboards/qvantum-content-kalender/`
- Het `linkedin-dashboard` is een APART Node.js project — raak het NIET aan
- Wijzig NOOIT bestanden buiten deze map
- Als functionaliteit overlapt met het LinkedIn dashboard: vraag eerst of het hier of daar thuishoort

## Qvantum brand toepassing
- **Primaire kleur:** Brown #91877A (headings, accenten, navigatie)
- **Tekst:** Off-black #232222
- **Achtergronden:** Off-white #F4F4F4, Beige #E5E6DF, White #FFFFFF
- **Accent (sparingly):** Navy #002656, Blue #3164FD
- **Font:** Host Grotesk (Regular 400, Medium 500, SemiBold 600, Bold 700)
- **Geen italics.** Gebruik font weight of kleur voor nadruk
- Referentie: de `qvantum-brand-guidelines` skill in Claude voor volledige specificaties

## Content planning context
De kalender dekt de volgende contentkanalen voor Qvantum:
- LinkedIn (bedrijfspagina + persoonlijke accounts)
- Social media (overige kanalen)
- Nieuwsbrieven / email campagnes
- Blog / website content
- Campagne-gerelateerde content (beurzen, productlanceringen)

De bijbehorende `marketing/media en item planning.xlsx` bevat aanvullende planningsdata — raadpleeg dit bestand voor context wanneer je content items toevoegt of wijzigt.

## Doelgroep
- Primair: Viktor (brand manager) voor planning en overzicht
- Secundair: Qvantum marketing team voor uitvoering
- De kalender moet visueel helder zijn: maand/weekoverzicht, kleurcodering per kanaal, status per item

## Bij wijzigingen
1. Open het HTML bestand in de browser en test visueel
2. Check of alle interactieve elementen werken (filters, toevoegen, bewerken)
3. Controleer dat de styling Qvantum brand-compliant is
4. Bestandsgrootte in de gaten houden — bij >200 KB overwegen om data te externaliseren
5. Maak een v2 kopie als je grote wijzigingen maakt (behoud altijd een werkende versie)

---
> Source: [dehanos78/qvantum-content-kalender](https://github.com/dehanos78/qvantum-content-kalender) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
