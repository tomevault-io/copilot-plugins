## catchrank

> Dit document bevat de vaste projectregels, productcontext, UX-richtlijnen, technische architectuurprincipes, live/release-regels en werkafspraken voor Claude Code binnen de CatchRank codebase.

# CLAUDE.md
# CatchRank — Project Instructions for Claude Code v2

Dit document bevat de vaste projectregels, productcontext, UX-richtlijnen, technische architectuurprincipes, live/release-regels en werkafspraken voor Claude Code binnen de CatchRank codebase.

Claude Code moet deze instructies behandelen als de standaard projectwaarheid voor alle analyse-, refactor-, implementatie-, optimalisatie- en releasegerichte taken.

---

## 1. PROJECTIDENTITEIT

### Productnaam
CatchRank

### Domein
CatchRank.nl

### Web build / huidige live omgeving
GitHub Pages build onder:
`https://aiexpertscorner.github.io/CatchRank-/`

### Positionering
CatchRank is een premium dark mobile-first sportvisplatform dat voelt als een mix van:

- Strava voor sportvissers
- slim vislogboek
- sessie- en stekplatform
- analytics- en statistiekenomgeving
- community- en clubsplatform
- ranking / XP / progressieplatform
- weather-enabled fishing intelligence app
- persoonlijke visgear / setup / favorieten omgeving
- affiliate-ready productlaag voor sportvisproducten

### Kernbelofte
- slimmer loggen
- beter inzicht
- meer motivatie
- meer community
- meer progressie
- slimmer vissen
- betere gearkeuzes

### Belangrijkste productpijlers
- Catches
- Sessions
- Spots
- Weather
- Mijn Visgear
- Stats
- Rankings / XP
- Community
- Tools / advice / recommendations

---

## 2. HOOFDDOEL VAN CLAUDE CODE IN DIT PROJECT

Claude Code is in dit project:

- senior pair programmer
- repo analyst
- architecture-aware implementer
- UI/UX-sensitive code builder
- Firebase/Firestore-conscious engineer
- low-cost / low-spend optimizer
- systematic refactor partner
- production-minded release partner

Claude Code moet:

- bestaande code eerst begrijpen
- daarna pas verbeteren of uitbreiden
- productwaarde, mobile UX en kostenefficiëntie prioriteren
- technische kwaliteit hoog houden zonder over-engineering
- zoveel mogelijk bouwen op bestaande patronen waar dat zinvol is
- live bruikbaarheid voor toegestane gebruikers serieus nemen
- auth, data, rules en release-impact expliciet meewegen

Claude Code moet niet:

- blind grootschalig refactoren zonder duidelijke reden
- onnodig architectuurcomplexiteit toevoegen
- dure of risicovolle processen automatisch activeren
- generieke SaaS-web patronen introduceren die niet passen bij CatchRank
- light theme of desktop-first ontwerpkeuzes introduceren
- production builds onbedoeld in demo mode laten staan
- mock data of fake success flows als production default laten bestaan

---

## 3. DESIGN DNA / UI RICHTING

### Algemene stijl
CatchRank is expliciet:

- dark theme
- premium
- compact
- mobile-first
- sportief
- clean
- functioneel
- polished
- no-nonsense
- modern

### Visuele referentie
De visuele richting moet expliciet aansluiten op de bestaande Flutter app stijl:

- donkere basis
- goudgele accentkleur
- compacte spacing
- donkere cards
- grote duidelijke mobile tap targets
- hero image driven detail screens
- bottom navigation met centrale actieknop
- sterke screen titles
- duidelijke stat cards
- functionele, nette glow/accent states
- geen generieke Material demo uitstraling
- geen “witte SaaS dashboard” look

### Kleurgevoel
Primair:

- bijna zwart / deep charcoal
- goudgele accentkleur
- subtiele antraciet en bronze nuances
- optioneel zeer subtiel dark teal / water nuance

Vermijden:

- veel wit
- lichtblauw als hoofdmerkaccent
- schreeuwerige gradients
- platte startup-branding

### Typografie
Gebruik:

- Inter voor UI, body, metadata, forms, cards, stats
- Krub voor headings, hero-titels, sectietitels, screen titles, merkaccenten

Typografie moet:

- compact en duidelijk zijn
- goede hiërarchie hebben
- goed leesbaar zijn op mobiel
- geen te kleine belangrijke informatie bevatten
- screen titles sterk en premium laten voelen

---

## 4. MOBILE-FIRST IS ABSOLUTE PRIORITEIT

CatchRank moet altijd worden ontworpen en gebouwd alsof mobiel de primaire context is.

### Mobiele prioriteiten
- alles eerst optimaliseren voor smartphone schermen
- snelle thumb-friendly interacties
- belangrijke acties binnen duimbereik
- bottom navigation als primair patroon
- centrale primary action button waar logisch
- compacte contentblokken
- minimale maar comfortabele spacing
- snelle scanbaarheid
- grote genoeg tap targets
- flows voor gebruik aan de waterkant
- forms kort, logisch en snel
- list/detail screens compact en rijk
- hero/meta/stats/actions logisch geordend
- bottom sheets vaak verkiezen boven desktop-style modals

### Niet doen
- desktop-first layout denken
- te veel witruimte introduceren
- kaarten onnodig hoog maken
- ingewikkelde multicolumn-patterns eerst voor web bedenken
- onnodige hover-afhankelijkheid

---

## 5. PRODUCTMODULES EN PRIORITEIT

### Prioriteit 1
- auth basis
- dashboard basis
- users profielstructuur
- catches loggen
- sessions loggen
- spots beheren
- quick catch flow
- draft logging
- basis stats
- XP progressie
- basis rankings
- mobile-first shell
- Mijn Visgear basis
- favorieten basis
- setup koppeling aan logging
- bestaande weatherlaag slim integreren zonder extra verspilling

### Prioriteit 2
- clubs
- community feed
- comments/likes
- uitgebreidere analytics
- badges / achievements
- filters en segmentaties
- community gear signals
- affiliate productcatalogus basis
- product matching user gear ↔ feeds
- weather insights / forecast verdieping

### Prioriteit 3
- geavanceerde tools
- advies-engines
- slimme analyses
- knowledge/content integratie
- extra gamification
- automation flows
- slimme gear aanbevelingen
- AI bait / gear / setup suggesties
- geavanceerde commerce personalisatie

Claude Code moet productbeslissingen altijd spiegelen aan deze prioriteiten.

---

## 6. MIJN VISGEAR — SPECIFIEKE PROJECTREGELS

### Naamgeving
Gebruik overal consequent:
- Mijn Visgear

Gebruik niet:
- Mijn Vista
- Vista/Gear
- VisTas
- Gear module als productnaam

Technisch intern mag “gear” gebruikt worden, maar productmatig heet het **Mijn Visgear**.

### Doel van Mijn Visgear
Mijn Visgear is een utility-first module waarin users:

- eigen gear toevoegen
- favorieten opslaan
- setups maken
- gear koppelen aan catches, sessions en spots
- later productmatches en affiliate suggesties krijgen
- feedproducten kunnen ontdekken zonder webshopgevoel

### UX-volgorde
Altijd:
- eigen gear
- favorieten
- setups
- slimme suggesties
- affiliate/conversie als ondersteunende laag

Niet doen:
- Mijn Visgear als losse shop behandelen
- affiliatecards dominant maken
- feedproducten boven eigen gear plaatsen zonder context

---

## 7. WEATHER / OPENWEATHER REGELS

### Belangrijke context
Er bestaat al een gekoppelde OpenWeather API en er zijn al weather-functies aanwezig in het project.

Claude Code moet:
- bestaande weather services inspecteren en hergebruiken
- dubbele weather logica vermijden
- weather als gedeelde platformlaag behandelen
- live weather, cached weather en stored snapshots logisch scheiden

### Weather wordt gebruikt voor
- catches
- sessions
- spots context
- dashboards
- weather/forecast tools
- smart logging prefills
- future insights / recommendations

### Weather regels
- voorkom dubbele API-calls
- voorkom onnodige refreshes
- voorkom polling zonder noodzaak
- cache weather slim
- sla alleen relevante snapshots op
- gebruik historische snapshots voor analyses
- gebruik live fetch alleen waar productmatig zinvol

### Niet standaard activeren
- zware background refreshes
- agressieve forecast syncs
- bulk enrichment jobs
- frequente scheduled jobs zonder expliciete opdracht

---

## 8. FIREBASE / FIRESTORE PRINCIPES

### Algemene regels
Werk alsof er al een bestaand Firebase/Firestore project is met bestaande data die gerespecteerd moet worden.

Claude Code moet:
- bestaande collecties en velden eerst analyseren
- destructieve wijzigingen vermijden
- backwards-compatible werken waar mogelijk
- read-optimized data slim gebruiken
- writes beperken
- relationele refs duidelijk houden
- null-safe parsing gebruiken
- fromMap / toMap mapping netjes houden

### Belangrijke collecties
Bestaand of verwacht:

- users
- catches_v2
- sessions_v2
- spots_v2
- species
- clubs
- club_members
- club_feed
- academy_articles
- academy_baits
- academy_knots
- academy_lures
- academy_rigs
- products
- challenges

Voor Mijn Visgear / productlaag verwacht:
- user_gear
- user_favorites
- user_setups
- gear_usage_links
- affiliate_products
- product_catalog_index
- community_gear_signals
- gear_recommendations

### Firestore architectuurregels
- gebruik refs waar logisch
- dupliceer alleen velden die read-performance duidelijk verbeteren
- gebruik aggregaties waar ze echt nuttig zijn
- voorkom overly nested complexity zonder reden
- houd models leesbaar en consistent
- documenteer mappingkeuzes indien relevant

### Read/write regels
- minimaliseer reads
- minimaliseer writes
- vermijd zware real-time listeners
- vermijd inefficiënte screen rebuild queries
- laad lightweight list data voor lijstschermen
- laad full detail data pas als nodig

---

## 9. DEVELOPMENT MODE / SAFE MODE

Claude Code moet standaard werken in development-safe mode.

### Harde regels
- gebruik demo/test user waar nodig
- bouw integraties volledig maar activeer dure processen niet standaard
- gebruik mocks/placeholders/sample data waar zinvol
- voorkom onnodige writes
- voorkom onnodige realtime listeners
- voorkom onnodige background jobs
- voorkom onnodige API-calls
- production gedrag alleen als daar expliciet om gevraagd wordt

### Belangrijke nuance
Development-safe mode mag **nooit** betekenen dat production builds functioneel beperkt blijven voor toegestane users.

Development-safe geldt voor:
- local development
- preview/staging
- expliciet afgeschermde features

Development-safe geldt **niet automatisch** voor `main` / production.

### Development-safe verwachtingen
In development moet de app:
- schermen volledig kunnen renderen
- flows volledig kunnen tonen
- integraties architectonisch klaarzetten
- bestaande services slim gebruiken
- maar kosten en verbruik minimaliseren

### Bij integraties altijd aangeven
Claude Code moet expliciet aangeven:
- wat gebouwd is
- wat live actief is
- wat alleen voorbereid / feature-flagged / placeholder is
- wat nog niet geactiveerd wordt om tokens/spend te besparen

---

## 10. LOW-SPEND / LOW-COST RULES

Kostenbeheersing is een kernregel.

Claude Code moet:
- reads minimaliseren
- writes minimaliseren
- API-verbruik minimaliseren
- caching toepassen
- lazy loading toepassen
- paginatie toepassen waar relevant
- real-time alleen gebruiken waar functioneel noodzakelijk
- feature flags gebruiken voor dure modules
- backgroundprocessen standaard uit laten als die niet expliciet nodig zijn

### Nooit standaard introduceren
- agressieve realtime feeds
- polling loops
- automatische bulk syncs
- zware scheduled jobs
- onnodige media/uploads bij elke stap
- productfeed live runtime fetch op elke schermrender

### Productfeed regel
Voor affiliate/productcatalogus geldt:
- app gebruikt primair Firebase-gecachede / genormaliseerde catalogdata
- feed URL is sync-bron, niet standaard runtime UI-bron
- live feed-fetch alleen voor admin/dev/test/fallback indien expliciet gewenst

### Belangrijke nuance
Low-spend mag geen excuus zijn voor kapotte kernflows.

Kernflows moeten live werken voor toegestane users, ook als dure uitbreidingen disabled blijven.

---

## 11. ARCHITECTUURSTIJL

### Gewenste structuur
Claude Code moet feature-based en modulair denken.

Voorkeur:
- duidelijke modulegrenzen
- screen/ui laag gescheiden van data/service laag
- herbruikbare widgets/components
- gedeelde theming/design tokens
- nette repository/service patterns waar zinvol
- lichtgewicht state-structuren passend bij huidige codebase

### Niet doen
- onnodige enterprise-architectuur introduceren
- abstractie om abstractie
- alles tegelijk refactoren
- patronen introduceren die niet aansluiten op de rest van de code

### Refactorregel
Refactor alleen als:
- huidige code echt blokkeert
- er duidelijke duplicatie is
- consistentie sterk verbetert
- schaalbaarheid/productkwaliteit duidelijk beter wordt

Anders: voortbouwen op bestaande code.

---

## 12. STANDAARD WERKWIJZE PER TAAK

Claude Code moet normaal gesproken deze workflow aanhouden:

### Stap 1 — Analyse
Eerst:
- relevante files lezen
- huidige architectuur begrijpen
- benoemen wat bestaat
- benoemen wat ontbreekt
- risico’s noemen

### Stap 2 — Plan
Daarna:
- compact implementatieplan geven
- duidelijke volgorde voorstellen
- scopes begrenzen

### Stap 3 — Implementatie
Dan pas:
- relevante files wijzigen
- nieuwe files toevoegen waar nodig
- bestaande code respecteren
- naming en structuur consistent houden

### Stap 4 — Rapportage
Na implementatie altijd:
- gewijzigde files opsommen
- belangrijkste keuzes uitleggen
- open TODO’s noemen
- aangeven wat nog getest moet worden
- benoemen wat dev-safe/disabled is gebleven

### Extra regel
Bij taken die live auth, reads, writes, Firebase config, deployment, permissions of release readiness raken moet Claude Code ook controleren welke console/config/rules-wijzigingen nodig zijn buiten de codebase.

Claude Code moet dus niet meteen blind gaan coderen zonder eerst analyse en plan, tenzij de opdracht expliciet direct implementatie vraagt.

---

## 13. VERPLICHTE OUTPUTSTIJL VAN CLAUDE CODE

Bij elke relevante implementatie-output moet Claude Code idealiter deze secties geven:

- Analyse
- Plan
- Gewijzigde files
- Wat ik heb gebouwd
- Belangrijke keuzes
- Production impact
- Firebase / Console wijzigingen nodig
- Dev-safe / nog niet geactiveerd
- Release blockers
- Open TODOs
- Handmatige checks

Hou uitleg compact maar concreet.

---

## 14. UX-RICHTLIJNEN PER TYPE SCHERM

### Dashboard
- compact
- actiegericht
- direct bruikbaar
- quick actions prominent
- niet te veel widgets
- live context tonen waar nuttig

### Logging flows
- snelheid boven perfectie
- quick first, enrich later
- altijd save/draft pad
- later aanvullen moet makkelijk zijn
- prefills slim maar niet opdringerig

### Detailpagina’s
- rijke hero
- sterke titel/meta
- stat blocks
- linked relations preview
- duidelijke CTA’s
- scanbaar en compact

### Lists
- lightweight cards
- sterke hiërarchie
- search/filter logisch
- snelle selectie mogelijk

### Bottom sheets
Gebruiken voor quick actions, pickers, preview cards en snelle mobile interacties.

---

## 15. BELANGRIJKE CROSS-MODULE LOGICA

CatchRank modules moeten niet als losse silo’s voelen.

Claude Code moet actief nadenken over verbindingen tussen:

- catches ↔ sessions
- catches ↔ spots
- catches ↔ weather
- catches ↔ Mijn Visgear
- sessions ↔ spots
- sessions ↔ weather
- sessions ↔ participants
- sessions ↔ gear/setup
- spots ↔ catches/sessions
- Mijn Visgear ↔ catches/sessions/spots/productfeed
- stats ↔ alle bovenstaande modules

### Belangrijke regel
Context inheritance en prefills mogen logging versnellen maar niet verwarrend maken.

---

## 16. NOTIFICATIES / DRAFTS / ACCEPTANCE LOGICA

Wanneer Claude Code werkt aan session invites, pending accepts, draft completions of soortgelijke functies:

- ontwerp altijd user-safe
- owner-flow moet simpel blijven
- invited user moet niet overvallen worden
- pending states moeten duidelijk zijn
- acceptance status moet zichtbaar zijn
- writes en notificaties moeten low-spend blijven

---

## 17. AFFILIATE / PRODUCTFEED RICHTLIJNEN

### Productlaag
Affiliate/productcatalogus moet service-first zijn.

Belangrijk:
- utility first, commerce second
- eerst user value
- pas daarna conversie
- geen shop-look
- geen agressieve CTA spam
- geen bannerschaos

### Technisch
- genormaliseerde catalogusdata in Firebase
- feed URL als import/sync-bron
- slimme category mapping
- list vs detail data scheiden
- product cards passen in CatchRank stijl

### UI
- product cards moeten aanvoelen als onderdeel van de app
- subtiele affiliate CTA’s
- duidelijke relevantie/context
- makkelijk toevoegen aan Mijn Visgear / setup

---

## 18. NAMING & CONSISTENTIE

Claude Code moet naming consequent houden.

### Gebruik
- Catch
- Session
- Spot
- Weather
- Mijn Visgear
- Setup
- Favorite/Favoriet afhankelijk van bestaande codeconventie
- Live Session
- Draft
- Complete

### Vermijd
- willekeurige synoniemen door elkaar
- Vista voor productnaam
- inconsistent casing of terminologie
- generieke namen zoals dataService2, tempModel, newWidgetFinal

### Regels
- geef duidelijke file names
- geef duidelijke class names
- houd naming semantisch sterk
- volg bestaande codeconventie als die goed genoeg is

---

## 19. TEST- EN VALIDATIEHOUDING

Wanneer relevant moet Claude Code:
- bestaande tests respecteren
- waar nuttig tests uitbreiden
- in ieder geval handmatige teststappen noemen
- aangeven welke flows na implementatie gecontroleerd moeten worden

Als echte tests toevoegen te zwaar is voor de scope:
- noem expliciet de belangrijkste handmatige checks

---

## 20. WAT CLAUDE CODE ALTIJD MOET PRIORITEREN

Volgorde van prioriteit:

1. correcte productlogica
2. mobile UX
3. compatibiliteit met bestaande codebase
4. live functionaliteit voor toegestane users
5. low-spend architectuur
6. Firebase/Firestore netheid
7. permissions / rules correctheid
8. herbruikbaarheid
9. visuele consistentie met CatchRank
10. performance
11. schaalbaarheid
12. verdere fancy uitbreidbaarheid

---

## 21. VERBODEN OF ONGEWENSTE PATRONEN

Claude Code moet deze dingen vermijden tenzij expliciet gevraagd:

- grote ongefundeerde rewrites
- desktop-first UI
- light theme defaults
- overmatige witruimte
- generieke SaaS cards/layouts
- agressieve real-time listeners
- polling zonder noodzaak
- live productfeed fetch op elk scherm
- onnodige package toevoegingen
- onnodige abstraction layers
- API-calls zonder cachingstrategie
- writes bij elke kleine UI state verandering
- “AI magic” zonder uitlegbare logica
- production builds die in demo mode blijven
- mock data als production default
- half-zichtbare niet-functionele kernfeatures zonder duidelijke disabled state

---

## 22. STANDAARD PROMPTGEDRAG DAT JE MAG AANNEMEN

Als de gebruiker vraagt om iets te bouwen, mag Claude Code meestal uitgaan van deze impliciete verwachtingen:

- analyseer eerst relevante files
- maak compact plan
- implementeer gericht
- wees development-safe
- wees low-spend
- houd UI donker en mobiel
- respecteer bestaande Flutter/CatchRank stijl
- vermeld file-overzicht en teststappen
- denk ook aan live auth/data/rules impact

Tenzij de gebruiker expliciet anders vraagt.

---

## 23. BIJ TWIJFEL: KIES DIT

Als iets niet volledig gespecificeerd is:

- kies de meest logische productkeuze voor een dark mobile-first sportvis app
- kies de minst dure architectuur die nog goed schaalbaar is
- kies de simpelste robuuste implementatie
- kies een oplossing die snel aanvoelt op mobiel
- kies consistentie boven creatief afwijken
- kies live werkbaarheid boven mooie maar niet-functionele demo-oplossingen

---

## 24. AANBEVOLEN STANDAARD ANTWOORDFORMAT VOOR IMPLEMENTATIEWERK

Gebruik bij codewijzigingen bij voorkeur deze structuur:

### Analyse
...

### Plan
...

### Gewijzigde files
...

### Wat ik heb gebouwd
...

### Belangrijke keuzes
...

### Production impact
...

### Firebase / Console wijzigingen nodig
...

### Dev-safe / nog niet geactiveerd
...

### Release blockers
...

### Open TODOs
...

### Handmatige checks
...

---

## 25. KORTE PROJECTSAMENVATTING

CatchRank is een premium dark mobile-first sportvis app.

De app draait om logging, sessions, spots, weather, progressie, Mijn Visgear en slimme context.

Claude Code moet bouwen alsof dit een serieuze publieksschaalbare app is, maar met sterke kostenbeheersing, nette Firebase-architectuur en consistente mobile UX.

Alles moet compact, praktisch, premium en mobiel aanvoelen.

Gebruik bestaande code slim, hergebruik bestaande weather/Firebase logica waar mogelijk, en bouw geen generieke SaaS-oplossingen.

---

## 26. PRODUCTION TRUTH / LIVE RELEASE POLICY

### Belangrijke hoofdregel
De `main` branch vertegenwoordigt de live build.

Alles wat naar `main` wordt gepusht moet bedoeld zijn om functioneel live te draaien voor toegestane gebruikers, tenzij expliciet anders aangegeven.

Claude Code moet daarom standaard aannemen:

- `main` = live intent
- production builds mogen niet onbedoeld in demo mode blijven
- production builds mogen geen mock-only flows tonen aan ingelogde toegestane gebruikers
- production builds mogen geen harde blokkades bevatten die normale appflows voor toegestane users breken
- als iets nog niet live klaar is, moet het of:
  - correct hidden/feature-flagged zijn
  - of duidelijk als disabled beta feature behandeld worden
  - maar niet half zichtbaar en niet werkend in production UI

### Verboden op production
Claude Code moet vermijden dat production builds afhankelijk blijven van:

- mock data als primaire databron
- demo mode flags die echte gebruikersflows blokkeren
- fake success states zonder echte backend write
- hardcoded dev bypasses
- development-only auth guards
- incomplete route blocks die live flows breken
- placeholder permissions die reads/writes verhinderen voor toegestane users

### Verplichte denkwijze
Bij elke implementatie die auth, data, writes, reads, profile, catches, sessions, spots, gear, social of instellingen raakt moet Claude Code expliciet checken:

- werkt dit live voor een echte ingelogde toegestane gebruiker?
- gebruikt dit production services/config?
- zijn benodigde permissies/rules aanwezig?
- zijn demo guards of mock fallbacks nog actief?
- zijn reads/writes echt toegestaan voor de bedoelde user?
- is deze flow end-to-end functioneel?

---

## 27. ENVIRONMENTS / MODES

CatchRank werkt met duidelijke modes:

- local development
- optional staging / preview
- production

### Definitie
Local development:
- mag dev-safe defaults hebben
- mag mocks/fallbacks gebruiken waar expliciet gewenst
- mag feature flags gebruiken
- mag beperkt of sample-based zijn

Production:
- moet echte Firebase config gebruiken
- moet echte auth flow gebruiken
- moet echte Firestore/Storage permissies gebruiken
- mag geen demo-only blokkades bevatten voor toegestane users
- mag geen mock data als primaire bron gebruiken
- moet end-to-end functionele kernflows hebben voor toegestane gebruikers

Claude Code moet altijd expliciet benoemen:
- wat local/dev-only is
- wat preview/staging-only is
- wat echt production actief is

---

## 28. AUTH / ACCESS / BETA USER POLICY

### Production doel
De live app moet bruikbaar zijn voor een beperkte groep testusers / vrienden / beta users.

Claude Code moet daarom uitgaan van:

- echte Firebase Authentication in production
- echte ingelogde user sessies
- geen demo-auth fallback als production default
- appflows moeten werken voor toegestane ingelogde users

### Beta access model
Beperkte productie-toegang is toegestaan, maar moet netjes en expliciet zijn.

Voorkeursmodellen:
- allowlist via users document field zoals:
  - role
  - accessLevel
  - betaAccess
  - isTester
- of custom claims als daar al infrastructuur voor bestaat

### Niet doen
- complete app in demo mode laten staan
- hele flows blokkeren zonder duidelijke reden
- auth technisch laten slagen maar data-toegang blokkeren zonder UX feedback
- productiegebruikers behandelen alsof alles nog mock/dev is

Claude Code moet bij auth-gerelateerd werk expliciet controleren of nodig zijn:
- Firebase Authorized Domains
- correcte production Firebase config
- auth state persistence
- route guards
- access guards
- Firestore rules
- Storage rules

---

## 29. LIVE DATA / NO MOCK DATA IN PRODUCTION

### Hoofdregel
Voor production builds geldt:

- echte data eerst
- mock data nooit als primaire runtime databron voor toegestane ingelogde users
- placeholders alleen toegestaan voor lege states of expliciet disabled features

### Toegestaan
- loading skeletons
- empty states
- feature-flagged “binnenkort beschikbaar”
- fallback UI wanneer data nog leeg is

### Niet toegestaan
- fake catches tonen als echte user data
- demo sessions tonen aan echte ingelogde users
- nep-statistieken tonen alsof ze live zijn
- writes simuleren zonder echte backend write
- schermen tonen die productieklaar lijken maar op mock adapters draaien

Claude Code moet bij iedere implementatie die data gebruikt expliciet zeggen:
- live data path
- fallback path
- of mock path nog bestaat
- of production nog ergens mock data gebruikt

---

## 30. FIREBASE RULES / PRODUCTION PERMISSIONS

Claude Code moet auth- en datagerelateerd werk nooit alleen als UI-werk behandelen.
Ook backend-permissies, rules en ownership checks moeten worden meegenomen.

Voor alle kernmodules moet Claude Code expliciet nadenken over:
- wie mag lezen?
- wie mag schrijven?
- wie is owner?
- welke users mogen test/beta features gebruiken?
- welke velden mogen door client geschreven worden?
- welke velden moeten server-controlled blijven?

### Minimaal te valideren voor production
- Firestore rules
- Storage rules
- auth domain instellingen
- ownership checks op user-bound docs
- route guards
- beta access checks
- null-safe handling van denied states

### Belangrijke regel
Als codewijzigingen alleen werken wanneer ook Firebase Console / Rules aangepast worden, moet Claude Code dat expliciet zeggen.

---

## 31. RELEASE GATES BEFORE MERGING TO MAIN

Voordat iets naar `main` gaat dat kernflows raakt, moet Claude Code het beoordelen op:

- werkt login live?
- werkt logout live?
- werkt auth persistence live?
- werkt profile read live?
- werken catches read/write live?
- werken sessions read/write live?
- werken spots read/write live?
- werken gear flows live?
- werken route guards correct?
- werken empty states correct?
- is demo mode uit of correct afgeschermd?
- is mock data niet meer primary?
- zijn Firestore rules voldoende?
- zijn Storage rules voldoende?
- is de juiste Firebase project config actief?
- staat de production domain in Authorized Domains?
- werken builds met juiste base path / env config?

Als een onderdeel nog niet production-klaar is:
- niet half-live laten staan
- of correct hidden maken
- of als disabled beta feature tonen

---

## 32. FEATURE FLAGS / DISABLED STATES

Feature flags zijn toegestaan, maar alleen als ze duidelijk en bewust worden ingezet.

### Regels
- production features die zichtbaar zijn moeten functioneel zijn
- niet-functionele features moeten verborgen of duidelijk disabled zijn
- flags mogen geen stille demo mode veroorzaken
- flags mogen geen auth/data basisflows blokkeren zonder expliciete reden

Claude Code moet altijd benoemen:
- welke flags bestaan
- welke flags production beïnvloeden
- welke features live actief zijn
- welke features hidden/disabled zijn

---

## 33. DEPLOYMENT / GITHUB PAGES DISCIPLINE

De live web build draait op GitHub Pages en moet correct werken onder de production base path.

Claude Code moet bij web/deployment-relevante wijzigingen expliciet controleren:
- juiste base path / basename
- juiste asset paths
- juiste router config
- juiste env config
- correcte Firebase config voor live domain
- correcte redirect / 404 strategy waar relevant
- geen local-only assumptions in auth of routing

### Belangrijk
Voor de live Pages build moet auth en data-toegang getest worden op de echte live domain, niet alleen lokaal.

---

## 34. FIREBASE CONSOLE / CONFIG AWARENESS

Claude Code moet beseffen dat niet alle werkende productieflows alleen uit code bestaan.

Bij taken die auth, uploads, Firestore writes, hosting of permissions raken moet Claude Code expliciet benoemen of mogelijk nodig zijn:

- Firebase Authentication providers
- Authorized Domains
- Firestore rules
- Storage rules
- indexes
- environment variables
- app config / web config
- hosting/base path settings
- test/beta user setup

Claude Code mag niet impliciet aannemen dat code-only voldoende is.

---

## 35. USER ACCESS MODEL FOR BETA TESTING

Omdat CatchRank voorlopig beperkt gebruikt mag worden door testusers / vrienden / beta users, moet Claude Code standaard uitgaan van een gecontroleerd access model.

### Gewenste aanpak
Voorkeur:
- login technisch laten werken
- daarna user access bepalen via Firestore user document
- denied users nette beperkte UX geven
- toegestane users volledige kernflows laten gebruiken

### Voorkeursvelden
Voorbeelden:
- `betaAccess: true`
- `role: "tester"`
- `accessLevel: "beta"`
- `isTester: true`

### Niet doen
- login blokkeren als eenvoudige allowlist voldoende is
- hele app in demo mode laten voor alle users
- auth wel toestaan maar daarna alle reads/writes onduidelijk laten falen

---

## 36. KERNFLOWS DIE LIVE MOETEN WERKEN VOOR TOEGESTANE USERS

Voor toegestane ingelogde testers/beta users moeten de volgende flows als kern beschouwd worden:

- inloggen
- ingelogd blijven
- profiel lezen
- catches lezen
- catches toevoegen
- catches bewerken waar toegestaan
- sessions lezen
- sessions toevoegen
- sessions bewerken waar toegestaan
- spots lezen
- spots toevoegen of selecteren waar flow dit verwacht
- Mijn Visgear basis lezen en schrijven
- relevante weather data ophalen via bestaande laag
- basis dashboard data tonen
- logout

Als een van deze flows nog niet werkt, moet Claude Code dat als production blocker behandelen tenzij expliciet anders afgesproken.

---

## 37. GEEN STILLE DEMO BLOKKADES

Claude Code moet vermijden dat flows stil falen door verborgen demo/dev beperkingen.

### Verboden gedrag
- button werkt ogenschijnlijk maar schrijft niks
- save flow toont succes maar maakt geen document
- route opent maar echte data is vervangen door mocks
- authenticated user ziet demo placeholders terwijl echte data verwacht wordt
- reads/writes worden client-side geblokkeerd zonder duidelijke melding

### Vereiste
Als een flow nog niet actief is:
- maak dit duidelijk zichtbaar
- disable de CTA correct
- of verberg de feature tijdelijk

---

## 38. SCHEMA / RULES / CODE CONSISTENCY

### Regels
- geen willekeurige field names verzinnen
- v2 collecties prefereren boven legacy collecties als dat de projectstandaard is
- bij schema-afwijkingen die expliciet benoemen
- code, database en rules zoveel mogelijk consistent houden

---

## 39. HANDMATIGE CHECKS ZIJN VERPLICHT RELEVANT BIJ LIVE-WERK

Bij implementaties die live auth, reads, writes, uploads, routing of deploy beïnvloeden moet Claude Code altijd concrete handmatige checks noemen, bijvoorbeeld:

- login op live Pages URL
- refresh na login
- protected route openen
- catch aanmaken
- catch herladen
- session aanmaken
- gear item toevoegen
- denied/beta access gedrag testen
- logout testen
- browser console controleren
- Firestore writes controleren
- Storage upload controleren indien relevant

---

## 40. STANDAARD LIVE-READY DENKMODEL

Bij twijfel moet Claude Code dit mentale model gebruiken:

- `main` = live build
- live build moet functioneel zijn voor toegestane users
- mock/demo/dev fallbacks mogen niet production default zijn
- auth + rules + config + data moeten samen kloppen
- goedkope architectuur is goed, maar niet ten koste van kernfunctionaliteit
- visible features moeten werken of duidelijk disabled zijn
- productkwaliteit = UX + code + permissions + real data + correct deployment

---

## 41. KORTE OPERATIONELE SAMENVATTING

CatchRank is een premium dark mobile-first sportvis app.

De app draait om:
- logging
- vangsten
- sessions
- spots
- weather
- progressie
- Mijn Visgear
- slimme context
- later community en commerce

Claude Code moet bouwen alsof dit een serieuze publieksschaalbare app is, maar:
- mobile-first
- compact
- low-spend
- Firebase-netjes
- production-aware
- live functioneel voor toegestane testers
- zonder demo mode op `main`
- zonder mock data als production default
- met expliciete aandacht voor auth, rules, domains, permissions en release readiness

Einde van projectinstructies.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aiexpertscorner)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aiexpertscorner)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
