## dcod-tapijt-studio-v2

> AI-gestuurde tapijt-dessin generator voor DCOD Printtapijt. Flask/Python backend,

# DCOD Dessinator

AI-gestuurde tapijt-dessin generator voor DCOD Printtapijt. Flask/Python backend,
vanilla JS frontend, Claude AI voor prompt-analyse. Doel: architecten en
interieurontwerpers laten binnen minuten een passend vloerconcept vinden.

## Rol van de gebruiker
Ab is niet-technisch en ondersteunt DCOD kosteloos als vriendendienst. Leg
technische keuzes kort en praktisch uit, geen jargon zonder toelichting.
Werktaal: Nederlands.

## Stack
- Backend: Python 3 / Flask (`app.py`, `modules_extra.py`)
- Frontend: `templates/index.html` (Dessinator), `templates/inspiratie.html`
  (conceptkeuze-flow), `static/js/app.js`
- AI: Anthropic Claude API (`claude-haiku-4-5-20251001`) voor promptanalyse
  en `/api/refine`
- Deployment: Render.com, auto-deploy vanuit GitHub
  (`appieoosterhof/dcod-tapijt-studio-v2`, branch `main`)
- API-sleutel staat in `api_key.txt` (root), in `.gitignore` — nooit
  hardcoden, nooit in bestandsnamen, nooit committen.

## BINDENDE ARCHITECTUURREGEL (5 juli 2026)
Geen nieuwe dessins meer als losse, handgeschreven SVG-tekenfunctie.
Alle nieuwe SVG's worden opgebouwd uit herbruikbare geometrische objecten
("Geometry Engine"): Lines, Arcs, Polygons, Noise, Offsets, Grid, Layers,
Masks, Rotation, Random. Eerste bouwsteen (Grid, in `geometry_engine/grid.py`)
staat al, volledig losstaand van de Dessinator — raakt `app.py`/
`modules_extra.py` niet aan totdat er bewust voor gekozen wordt iets te
integreren. Bestaande dessins mogen geleidelijk migreren, geen big-bang-herbouw.

## Vier deelsystemen
1. **Geometry Engine** — het brein, generieke patroonbouwstenen (in opbouw,
   losstaand in `geometry_engine/`)
2. **Dessinator** — de gebruikersinterface (grotendeels klaar)
3. **Mockup Engine** — "Bekijk in ruimte", perspectief-projectie via CSS
   matrix3d. Config in `static/js/app.js` (`ROOM_MOCKUPS`-object). Bekend
   probleem: perspectief klopt niet op de huidige kantoorfoto (FR-hoekpunten
   moeten opnieuw gekalibreerd worden) — bestond al vóór recente wijzigingen.
4. **Productie Engine** — SVG naar printklaar bestand. `svg_to_png()` in
   `app.py` is momenteel een STUB (doet niets, retourneert alleen het pad
   zonder te renderen) — nog te bevestigen of dit een echt probleem is en
   zo nodig te repareren.

## Een nieuw dessin toevoegen (5 verplichte stappen in app.py)
Missen van een van deze stappen laat de generator stilletjes terugvallen op
het generieke geometrische fallback-patroon:
1. Generator-functie (bij voorkeur via Geometry Engine-bouwstenen)
2. Registratie in `STYLE_GENERATORS`
3. Toevoegen aan de `extra_styles`-lijst
4. Routing-guards in `build_tile_svg` — gebruik `analysis["_prompt"]`
   (origineel), NIET `description_nl` (AI-omschrijving kan stijl-keywords
   bevatten die de routing per ongeluk overschrijven)
5. `label_map` voor de weergavenaam

## Bekende valkuilen
- **Keyword-botsingen**: Nederlandse woorden die toevallig als substring in
  een andere stijl-keyword-lijst voorkomen (bv. "bloem" matcht binnen
  "bloemrijk") kunnen verkeerde routing triggeren. Check altijd op
  substring-overlap bij nieuwe keywords.
- **SVG + Safari**: nooit `clipPath` gebruiken — breekt zodra de SVG als
  base64 in een `<img>`-tag wordt geladen.
- **Naadloze tegeling**: tegel is altijd 400px; motiefschaal via n×n
  tegeling waarbij n een deler van 400 moet zijn.
- **defs-hoisting**: als een tegel een `<defs>`-blok bevat (bv. ingebedde
  foto), wordt dat in `build_tile_svg`/`build_repeat_svg` automatisch 1x
  naar de buitenste defs verplaatst i.p.v. per kopie gedupliceerd — bestaat
  al, hergebruiken bij toekomstige raster-gebaseerde generators.
- **Cache-stamp**: bij elke wijziging aan `app.js` de `?v=` cache-stamp in
  de HTML-templates verhogen, anders laadt de browser een oude versie.

## Werkwijze
- Kleine, geïsoleerde, direct testbare stappen. Altijd een backup/commit
  vóór een risicovolle wijziging.
- Laat bij twijfel over scope of aanpak een korte vraag stellen in plaats
  van aannames te doen — vooral bij dingen die de live Dessinator kunnen
  breken.
- `git commit` = lokaal, veilig. `git push` = triggert Render auto-deploy,
  gaat DIRECT live. Vraag altijd expliciet toestemming vóór een push.
- Test in een privé-browservenster bij twijfel over cache-problemen.

## Openstaande punten
- Hotels & Leisure → Hotels & Hospitality hernoemen (projectkaart, paneel-ID
  `panel_hotels_en_leisure`, `DCOD_PROJECTS`-mapping in app.js)
- Floorvisualizer-perspectief kalibreren op de kantoorfoto
- Productie Engine (PNG-export) — bevestigen of `svg_to_png` echt niet-
  functioneel is, zo nodig repareren
- Geometry Engine verder bouwen: Lines, Arcs, Polygons, Noise, Offsets,
  Layers, Masks, Rotation, Random (Grid staat al)
- "Ontwerp zelf met AI" als aparte tweede ingang (vrije tekst stuurt
  bestaande parameters bij, zoals `/api/refine` nu al doet voor Aardlagen
  met `_al_lw_factor` — geen nieuw dessin laten verzinnen)

---
> Source: [appieoosterhof/dcod-tapijt-studio-v2](https://github.com/appieoosterhof/dcod-tapijt-studio-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
