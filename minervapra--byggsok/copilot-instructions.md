## byggsok

> > Kontekst for AI-assistenter som jobber med dette repositoriet.

# CLAUDE.md — Byggsok

> Kontekst for AI-assistenter som jobber med dette repositoriet.
> Oppdater denne filen når prosjektet endres.

## Prosjektoversikt

**Byggsok** er en hypergraf over norsk byggesakslovgivning. Systemet kobler
tolkningsuttalelser, lovtekster, forskrifter, forarbeider, rundskriv og
sivilombudet-uttalelser til paragrafer i plan- og bygningsloven (PBL),
Byggesaksforskriften (SAK10) og Byggteknisk forskrift (TEK17).

**Repository**: `Minervapra/Byggsok`
**Hovedbranch**: `claude/claude-md-mlggpmni6yvbnv90-NmAkK` (ikke `master` eller `main`)

> **VIKTIG:** Dette repositoriet har ingen `master`/`main`-branch.
> All merge og PR-er skal bruke `claude/claude-md-mlggpmni6yvbnv90-NmAkK` som base.
> Sjekk alltid `git branch -r` og `git remote show origin` før merge-operasjoner.

### Oppstart — les disse filene først

Når du starter en ny sesjon, les alltid:

1. **`TASKS.md`** — Konsolidert oppgaveoversikt med prioriteringer,
   metrikker og status for alle aktive oppgaver. Start her.
2. **`PICKUP_PROMPT.md`** — Ferdigskrevet prompt for å starte neste
   sesjon med riktig kontekst. Brukes ved sesjonsbytte.
3. **`tasks/lessons.md`** — Lessons learned. Unngå kjente feil.
4. **`DESIGN_PRINCIPLES.md`** — Arkitekturprinsipper for kontekststyring,
   søk og informasjonsflyt. Referansedokument ved arkitekturbeslutninger.

### Avslutning — oppdater ALLTID disse filene etter utført arbeid

Etter at en oppgave er ferdigstilt og committet, oppdater ALLTID:

1. **`PICKUP_PROMPT.md`** — Oppdater med: hva som ble gjort, ny status,
   oppdaterte nøkkeltall, nye lessons learned.
2. **`TASKS.md`** — Marker oppgaven som ferdig, oppdater metrikker.
3. **`tasks/lessons.md`** — Legg til nye lessons learned hvis relevant.
4. **`research/retrieval-gap-analyse.md`** — Logg retrieval-eksperimenter
   her når du svarer på brukerspørsmål. Dokumenter hvilke dokumenter
   som ble funnet/ikke funnet, rotårsaker (R1-R6) og hvilke
   reformuleringer som traff. Se Case-malen i filen.

### Nøkkeltall

| Metrikk | Verdi |
|---------|-------|
| Noder i hypergraf | 2 329 |
| Hyperkanter | 1 460 |
| Lovparagrafer dekket | 489 (PBL 252, SAK10 104, TEK17 133) |
| Lovparagrafer med dokumenter | 264 |
| Lovparagrafer med full_text | 477 (av 489) |
| Lovparagrafer med tema-tags | 325 (66.5%) |
| Lovtekst-filer | 72 (PBL 36, SAK10 18, TEK17 18) |
| Tolkningsuttalelser | 627 (KDD 368, Sivilombudet 259) |
| Rettspraksis | 54 (HR 20, Lagmannsrett 16, RT 18) |
| Forarbeider | 7 proposisjoner |
| Rundskriv | 21 (10 gjeldende, 10 delvis oppdatert, 1 SPR) |
| Temaer | 35 |
| Strukturerte innholdsfiler | ~155 |

## Spørsmålsruting — VIKTIG

Når en bruker stiller et spørsmål om byggesak i naturlig språk:

1. **Trestegs-klassifisering — du gjør det selv.**
   Du (Claude Code) er allerede en LLM og kan klassifisere direkte.
   Ingen API-nøkkel trengs. Bruk dette workflowet (~2800-3500 tokens):

   ```python
   from byggsok.classifier import (
       build_chapter_context, parse_chapters,
       build_section_detail, read_section_texts,
       parse_classification, theme_context, parse_themes,
       classify_source_priority,
   )
   from byggsok.hypergraph_query import HypergraphQuery

   # Steg 1: Hent kapitteloversikt (~1175 tokens)
   chapter_ctx = build_chapter_context()
   themes = theme_context()

   # Les chapter_ctx og identifiser:
   #   a) 3-5 relevante kapitler: [("pbl", 1), ("pbl", 12), ("sak10", 4)]
   #   b) 1-3 relevante temaer: ["strandsone", "soknadsplikt"]

   # Steg 2: Hent berikede §§ for valgte kapitler (~500-800 tokens)
   section_detail = build_section_detail([("pbl", 1), ("pbl", 12)])

   # Les section_detail og velg 1-8 spesifikke paragrafer ELLER
   # bokstav-undernoder. For brede §§ som §20-1 og SAK10 §4-1 vises
   # bokstaver med valgbare IDer (f.eks. lov:pbl§20-1-k for
   # terrenginngrep). BRUK ALLTID bokstav-ID når spørsmålet passer
   # en spesifikk bokstav — dette gir mye bedre presisjon i søket.

   # Steg 2b: Bestem kildeprioritering
   section_ids = parse_classification('["lov:pbl§20-1-k", "lov:sak10§4-1-f"]')
   priority = classify_source_priority(section_ids)
   # priority["forskrift_primary"] = True → forskrift + veiledning er hovedkilden
   # priority["include_guidance"] = True → kun True når forskrift er primær
   # priority["include_commentary"] = True → KDD lovkommentar for PBL plandel
   # priority["max_chars"] = 20000 (forskrift-primær) / 12000 (PBL-primær)
   # priority["guidance_max_chars"] = 5000 per § (kun når primær)

   # Steg 3: Les full lovtekst for valgte §§
   # Forskrift-primær: 20k tegn med veiledning (~5k tokens)
   # PBL-primær: 12k tegn med lovkommentar (~3k tokens)
   law_text = read_section_texts(
       section_ids,
       include_guidance=priority["include_guidance"],
       include_commentary=priority["include_commentary"],
       guidance_max_chars=priority["guidance_max_chars"],
       max_chars=priority["max_chars"],
   )

   # Steg 4: Slå opp i hypergrafen med de klassifiserte §§
   theme_ids = parse_themes('["rammetillatelse", "reguleringsplan"]')
   hg = HypergraphQuery()
   results = hg.resolve_question_broad(
       section_ids, theme_ids,
       keywords=["nøkkelord", "fra", "spørsmålet"],
   )
   ```

   **Steg 1** viser Del → Kap-hierarkiet med kuraterte nøkkelord per
   kapittel. Nøkkelordene disambiguerer kapitler (f.eks. «felles
   plan+byggesak/rammetillatelse(§1-7)» i Kap 1) og lar deg velge
   riktige kapitler på tvers av plandel og byggesaksdel.

   **Steg 2** viser alle §§ i de valgte kapitlene med titler,
   kryssreferanser (§-refs fra lovteksten) og nøkkelord (f.eks.
   «vesentlige virkninger», «plankrav»). **PBL kap 1 (Fellesdelen)
   inkluderes alltid automatisk** — den inneholder tverrgående
   bestemmelser (§ 1-6 tiltaksbegrep, § 1-8 strandsone, § 1-9
   klage) som rammer inn hele loven. Du trenger ikke velge den
   eksplisitt i steg 1. For brede paragrafer (§ 20-1, SAK10
   § 4-1 m.fl.) vises også bokstav-undernoder med valgbare IDer.
   **Foretrekk alltid bokstav-ID** fremfor hele paragrafen når
   spørsmålet passer en spesifikk bokstav. F.eks.
   `lov:pbl§20-1-k` (terrenginngrep) istedenfor `lov:pbl§20-1`
   (alle søknadspliktige tiltak). Dette reduserer kandidatpoolen
   dramatisk (f.eks. 7 dokumenter vs. 257).

   **Steg 2b** analyserer de klassifiserte §§ med
   `classify_source_priority()` og avgjør om SAK10/TEK17 er
   primærkilden. Når forskrift er primær, får veiledning større
   token-budsjett (5000 tegn/§) og forskriftsteksten bør dominere
   konteksten. Se «Forskrift-primær: Når SAK10/TEK17 dominerer» under.

   **Steg 3** leser selve lovteksten for de valgte §§. Med
   `include_guidance=True` inkluderes DiBK-veiledningen for
   SAK10/TEK17-§§ — dette gir forskriftskravet OG den praktiske
   tolkningen i ett steg. PBL-§§ får aldri veiledning.
   Token-budsjett: ~1000-4000 avhengig av prioritet.

   **Steg 4** bruker `resolve_question_broad()` som før, med tre
   parallelle oppdagelseskanaler:

   1. **Graf-traversering** — følger hyperkanter fra klassifiserte §§
   2. **Paragraf-referansesøk** — finner dokumenter som nevner §§ i tekst
   3. **Keyword-tekstsøk** — søker med synonym-ekspansjon

   Scorer er additive. Gap-basert cutoff returnerer 5-15 dokumenter.

2. **Bruk resultatene.** Hvert dokument har `relevance_score` og
   `matched_sections`. Baser svaret på de 3-5 høyest rangerte.
   Du har nå også lovteksten fra steg 3 — bruk den til å gi
   presise juridiske svar.

3. **Print ALT innhold fra resolve_question_broad().**
   Funksjonen returnerer tier-inndelte resultater (~5-14k tokens totalt):
   - Tier 1 (topp 5): full_content (maks 8k tegn/doc)
   - Tier 2 (neste 5): text_snippet (~3k tegn/doc)
   - Tier 3 (siste 5): bare metadata

   Print alt i én operasjon — **ikke kast innhold**:

   ```bash
   python3 -c "
   from byggsok.hypergraph_query import HypergraphQuery
   hq = HypergraphQuery()
   results = hq.resolve_question_broad(...)
   for i, r in enumerate(results):
       print(f'\n--- {i+1}. [score={r.get(\"relevance_score\",0)}] {r[\"id\"]} ---')
       print(r.get('label',''))
       content = r.get('full_content') or r.get('text_snippet') or ''
       if content:
           print(content)
   "
   ```

   Dette koster ~5-14k tokens — uproblematisk for hovedkonteksten.

4. **IKKE bruk subagenter for spørsmålsoppslag.** Resultatene fra
   `resolve_question_broad()` er for store for subagent-output
   (~25k token-grense). Kjør alltid oppslaget direkte med Bash i
   hovedkonteksten.

5. **Ikke hopp over klassifiseringen.** Direkte keyword-søk (`hg.search()`)
   gir for bred treffmengde og mangler juridisk forståelse. Trestegs-
   klassifiseringen forstår sammensatte spørsmål og leser lovteksten.

6. **Fallback.** Hvis klassifiseringen returnerer tom liste,
   bruk `hg.search(text=...)` som reserve. `resolve_question_broad()`
   inkluderer alltid bred tekstsøk og faller tilbake automatisk.

7. **Oppgi ALLTID referanser sammen med svaret.**
   Etter at du har gitt svaret på brukerens spørsmål, legg til en
   strukturert referanseseksjon med:

   **a) De 8 mest relevante dokumentene** (sortert etter score), med:
   - Dokument-ID og tittel/label
   - Relevans-score
   - Direkte lenke (URL fra dokumentets metadata)
   - Kort beskrivelse av hva dokumentet omhandler

   **b) Relevante lovbestemmelser** som er brukt i vurderingen:
   - Paragrafnummer og tittel
   - Kort forklaring av bestemmelsens relevans for spørsmålet

   Formater slik:

   ```
   ### Referanser

   #### Dokumenter
   1. **[score=45]** KDD 22/4570-2 — *Om bruksendring i boenhet*
      [regjeringen.no](https://www.regjeringen.no/...)
   2. ...
   (opptil 8 dokumenter)

   #### Lovbestemmelser
   - **PBL § 20-1** — Søknadspliktige tiltak (relevans: ...)
   - **TEK17 § 1-3** — Definisjon av boenhet (relevans: ...)
   ```

   **VIKTIG:** Aldri utelat referansene. Brukeren trenger disse for å
   verifisere og gå videre med kildene selv.

8. **Logg retrieval-eksperimenter.**
   Etter å ha besvart et brukerspørsmål, gjør en rask vurdering: fantes
   det relevante dokumenter som `resolve_question_broad()` burde ha
   funnet men ikke fant? Kjør 2-3 reformulerte søk for å sjekke.
   Logg funn til `research/retrieval-gap-analyse.md` med Case-malen.
   Dette bygger datagrunnlaget for systematiske søkeforbedringer.

## Forskrift-primær: Når SAK10/TEK17 dominerer konteksten

SAK10 og TEK17 er forskrifter med detaljerte operative regler. Når et
spørsmål handler konkret om det disse forskriftene regulerer, er
forskriftsteksten + DiBK-veiledning den definitive kilden. KDD-uttalelser
og annen forvaltningspraksis er da supplement — ikke hovedkilde.

`classify_source_priority()` oppdager dette automatisk, men du bør
også forstå logikken for å gjøre gode vurderinger:

### TEK17 er primærkilde når spørsmålet handler om:
- **Tekniske krav**: brannkrav, energikrav, konstruksjonssikkerhet,
  ventilasjon, lydkrav, fuktsikring, radon
- **Planløsning og tilgjengelighet**: boenhet-krav, universell utforming,
  trapp/balkong, bod/oppbevaring
- **Beregning og måling**: BRA, BYA, grad av utnytting, etasjehøyde,
  gesimshøyde, avstandsberegning
- **Sikkerhet**: naturfare, flom, skred, sikkerhet ved brann
- **Installasjoner**: vann, avløp, heis, elektro

**NB**: TEK17 er IKKE relevant bare fordi PBL § 29-4 (avstand til
nabogrense) nevnes. § 29-4 handler om plassering/avstand — dette er
en PBL-vurdering. TEK17 er kun relevant for § 29-4 dersom spørsmålet
gjelder *tekniske krav* (f.eks. brannmotstand mellom bygg).

### SAK10 er primærkilde når spørsmålet handler om:
- **Unntak fra søknadsplikt** (kap. 4): "Kan jeg bygge X uten å søke?",
  garasje, uthus, tilbygg, levegg, terrenginngrep, avstandskrav for
  unntatte tiltak
- **Søknadskrav og prosess** (kap. 2, 5): hva som krever søknad,
  søknadsdokumentasjon, nabovarsling, dispensasjon i søknad
- **Tiltak uten ansvarsrett** (kap. 3): hva tiltakshaver kan gjøre
  selv, mindre tiltak på bebygd eiendom
- **Tidsfrister** (kap. 7): 3-ukersfrist, 12-ukersfrist, beregning,
  konsekvens av oversittelse
- **Ferdigstillelse** (kap. 8): ferdigattest, midlertidig brukstillatelse,
  dokumentasjonskrav
- **Tiltaksklasser og kvalifikasjonskrav** (kap. 9, 11): inndeling i
  klasser, utdannings-/erfaringskrav per funksjon
- **Kontroll** (kap. 14): uavhengig kontroll, obligatorisk kontroll,
  kontrollplan

### Når forskrift er primær — slik bruker du det:
1. Les forskriftstekst + veiledning grundig (steg 3 med guidance)
2. Baser svaret PRIMÆRT på forskriftsteksten og veiledningen
3. Bruk KDD-uttalelser fra steg 4 kun som supplement/bekreftelse
4. Henvis primært til forskriftsbestemmelsen, sekundært til uttalelser

### Når forskrift IKKE er primær:
- Dispensasjonsvurderinger (PBL § 19-2) — skjønnsmessig, trenger praksis
- Planspørsmål — reguleringsplan, kommuneplan, arealformål
- Ulovlighetsoppfølging — PBL kap. 32, trenger forvaltningspraksis
- Eksisterende byggverk — PBL § 31-2 ff., skjønnsmessig
- Strandsonespørsmål — PBL § 1-8, tung rettspraksis

## Repositorystruktur

```
Byggsok/
├── CLAUDE.md                    # AI-assistentkontekst (denne filen)
├── TASKS.md                     # Oppgaveoversikt og status
├── hypergraph.json              # Generert hypergraf (~2.9 MB)
├── byggsok/                     # Python-pakke
│   ├── classifier.py            # LLM-klassifisering (spørsmål → §§)
│   ├── hypergraph_query.py      # Query-lag over hypergrafen
│   ├── cli.py                   # CLI-grensesnitt
│   ├── __init__.py
│   └── __main__.py
├── scripts/
│   ├── build_hypergraph.py      # Bygger hypergraph.json (inkl. fulltext-berikelse)
│   ├── diagnose_hypergraph.py   # Diagnoseverktøy for dekningsgrad
│   ├── update_law_chapters.py   # Oppdaterer kapittel-MD med Lovdata-tekster
│   └── convert_sivilombudet.py  # Konverterer sivilombudet-data
├── data/                        # Strukturert innhold
│   ├── pbl_struktur.yaml        # PBL paragrafstruktur (236 §§)
│   ├── sak10_struktur.yaml      # SAK10 paragrafstruktur (100 §§)
│   ├── tek17_struktur.yaml      # TEK17 paragrafstruktur (133 §§)
│   ├── lovdata_cache/           # Lovdata-seksjoner (autoritativ kilde)
│   │   ├── pbl_sections.json    # 247 PBL-paragrafer fra Lovdata
│   │   ├── sak10_sections.json  # 104 SAK10-paragrafer fra Lovdata
│   │   └── tek17_sections.json  # 133 TEK17-paragrafer fra Lovdata
│   ├── lov/                     # Lovtekster (pbl, nml) — oppdatert fra Lovdata
│   ├── forskrift/               # Forskriftstekster (sak10, tek17) — oppdatert fra Lovdata
│   ├── forarbeider/             # Proposisjoner
│   ├── rundskriv/               # Rundskriv
│   ├── rettspraksis/            # Rettspraksis (36 dommer, HR + LG)
│   ├── tolkningsuttalelser/kdd/ # Kuraterte KDD-uttalelser
│   └── incoming/                # Rå ustrukturert data (se under)
├── archive/                     # Arkiverte/utdaterte filer
│   └── legacy-raadata/          # Rå tolkningsuttalelser (migrert til data/)
└── research/                    # Research-rapporter
    └── retrieval-gap-analyse.md # Eksperiment-logg for retrieval-kvalitet
```

### data/incoming/ — Ustrukturert innhentet data

`data/incoming/` inneholder **rå data** hentet fra offentlige rettskilder
som ennå ikke er strukturert til repositoriets format (markdown med
YAML-frontmatter). Denne mappen er et mellomledd:

```
Offentlig kilde → data/incoming/{kilde}/ → strukturering → data/{målmappe}/
```

**Innhentingsprompter** for hver kilde finnes i
`data/incoming/INNHENTINGSPROMPTER.md`. Prioritert rekkefølge:

1. Prop. 115 L (2024-2025) — nyeste lovendring, 48 PBL-§§
2. KDD Lovkommentar plandelen — gratis kommentarutgave, ~150 §§
3. Nye KDD-uttalelser 2024-2025
4. Nye Sivilombudet-uttalelser 2024-2025
5. DiBK veiledning TEK17 — alle 133 §§
6. DiBK veiledning SAK10 — alle 104 §§

Se `research/rettskilder-analyse-2026-03-15.md` for full analyse.

## Teknisk stack

| Komponent | Teknologi |
|-----------|-----------|
| Språk | Python 3 |
| Dataformat | JSON (hypergraf), YAML (strukturfiler), Markdown (innhold) |
| LLM | Claude Haiku via Anthropic SDK |
| Avhengigheter | `pyyaml`, `anthropic` |

## CLI-kommandoer

```bash
# Slå opp en paragraf
python -m byggsok.cli section 29-4

# Still et spørsmål (krever ANTHROPIC_API_KEY)
python -m byggsok.cli ask "Kan jeg bygge garasje uten å søke?"

# Vis tema
python -m byggsok.cli theme dispensasjon

# Søk med filtre
python -m byggsok.cli search --section 29-4 --theme avstand-hovedregel

# Vis dokument (med utdrag)
python -m byggsok.cli doc 24_797

# Vis dokument (fullt innhold)
python -m byggsok.cli doc 24_797 --full

# Vis kapittel
python -m byggsok.cli chapter pbl-20

# Relaterte dokumenter
python -m byggsok.cli related 24_797

# Paragrafer uten tolkningsuttalelser
python -m byggsok.cli gaps

# Statistikk
python -m byggsok.cli stats
```

## Konvensjoner

- **Språk i kode**: Engelsk (variabelnavn, docstrings). Norsk i brukervendt output.
- **Node-IDer**: `lov:pbl§29-4`, `dok:data:kdd-2024-24-797`, `tema:dispensasjon`, `begrep:garasje`
- **Relasjonstyper**: hjemmel, utfyller, veileder, tolker, forarbeid-til, endret-ved, relatert
- **Filformat**: Markdown med YAML-frontmatter for innholdsfiler
- **Commit-meldinger**: Norsk, beskrivende, forklarer *hvorfor*

## AI-assistentretningslinjer

1. **Les før du skriver** — Les eksisterende filer før du foreslår endringer.
2. **Minimale endringer** — Bare det som trengs for oppgaven.
3. **Følg eksisterende mønstre** — Match stil og struktur i kodebasen.
4. **Klassifiser først** — Ved byggesaksspørsmål, bruk alltid `resolve_question()`.
5. **Oppdater denne filen** — Ved nye mønstre eller avhengigheter.
6. **Ingen hemmeligheter** — Aldri commit API-nøkler. Bruk miljøvariabler.
7. **Les lovteksten ved juridisk arbeid** — Når du klassifiserer dokumenter,
   tagger paragrafer, eller gjør annet arbeid som avhenger av lovens innhold:
   les ALLTID den faktiske lovteksten (`data/lov/`, `data/forskrift/`,
   eller `read_section_texts()`) istedenfor å stole på hukommelse eller
   tidligere prompts. Feil definisjoner i sesjon 13 førte til systematiske
   feil-klassifiseringer som måtte rettes opp.

---

## Workflow Orchestration

### 1. Plan Node Default

- Enter plan mode for any non-trivial task (three or more steps, or involving architectural decisions).
- If something goes wrong, stop and re-plan immediately rather than continuing blindly.
- Use plan mode for verification steps, not just implementation.
- Write detailed specifications upfront to reduce ambiguity.

### 2. Subagent Strategy

- Use subagents liberally to keep the main context window clean.
- Offload research, exploration, and parallel analysis to subagents.
- For complex problems, allocate more compute via subagents.
- Assign one task per subagent to ensure focused execution.

### 3. Self-Improvement Loop

- After any correction from the user, update `tasks/lessons.md` with the relevant pattern.
- Create rules for yourself that prevent repeating the same mistake.
- Iterate on these lessons rigorously until the mistake rate declines.
- Review lessons at the start of each session when relevant to the project.

### 4. Verification Before Done

- Never mark a task complete without proving it works.
- Diff behavior between main and your changes when relevant.
- Ask: "Would a staff engineer approve this?"
- Run tests, check logs, and demonstrate correctness.

### 5. Demand Elegance (Balanced)

- For non-trivial changes, pause and ask whether there is a more elegant solution.
- If a fix feels hacky, implement the solution you would choose knowing everything you now know.
- Do not over-engineer simple or obvious fixes.
- Critically evaluate your own work before presenting it.

### 6. Autonomous Bug Fixing

- When given a bug report, fix it without asking for unnecessary guidance.
- Review logs, errors, and failing tests, then resolve them.
- Avoid requiring context switching from the user.
- Fix failing CI tests proactively.

## Task Management

1. **Plan First** — Write the plan to `tasks/todo.md` with checkable items.
2. **Verify Plan** — Review before starting implementation.
3. **Track Progress** — Mark items complete as you go.
4. **Explain Changes** — Provide a high-level summary at each step.
5. **Document Results** — Add a review section to `tasks/todo.md`.
6. **Capture Lessons** — Update `tasks/lessons.md` after corrections.

## Core Principles

- **Simplicity First** — Make every change as simple as possible. Minimize code impact.
- **No Laziness** — Identify root causes. Avoid temporary fixes. Apply senior developer standards.
- **Minimal Impact** — Touch only what is necessary. Avoid introducing new bugs.

---

## Autoresearch — Autonom Rettskildeforbedringsloop

> **Status**: Fase 0-1 implementert, fase 2-3 gjenstår
> **Branch**: `autoresearch/hypergraf-forbedring`
> **Base**: `claude/claude-md-mlggpmni6yvbnv90-NmAkK`
> **Inspirert av**: [karpathy/autoresearch](https://github.com/karpathy/autoresearch)
> og [davebcn87/pi-autoresearch](https://github.com/davebcn87/pi-autoresearch)

### Designprinsipper

**1. Grafen er søkeruten — uten den drukner du i støy.**

Ren tekstsøk gir 95% recall men 8% precision. 400+ dokumenter uten
struktur. Grafen er trakten som snevrer ∞ mulige spørsmål ned til
5-15 presise kilder via: spørsmål → 1-8 §§ → graftraversering →
RRF-fusjon → Eckhoff-diversitetsinjeksjon.

**2. Eckhoff-hierarkiet styrer hva som er «godt nok».**

En § med bare tolkningsuttalelser (KDD) gir forvaltningens syn, men
mangler lovgiverviljen (forarbeider) og domstolenes tolkning
(rettspraksis). Eckhoff-dybde per § er primærmetrikken. Målet er
≥3 kildetyper for flertallet av §§.

**3. Loopen bygger relasjoner — ikke bare indekserer innhold.**

Autoresearch-loopen handler IKKE om å legge til flere dokumenter.
Den handler om å bygge *koblinger* mellom eksisterende dokumenter
og paragrafer. En forarbeid-proposisjon som er koblet til et
kapittel (`kap:pbl-19`) må resolves til spesifikke §§. En
HR-dom som nevner `§ 29-4` i teksten trenger en eksplisitt
hyperkant. Uten kanten finner søkeruten den ikke.

Prioritet for loopen:
1. **Parse §-referanser** i eksisterende dokumenttekst → nye kanter
2. **Resolve kapittel → §** for kilder koblet på kapittelnivå
3. **Kryssreferanse-resolusjon** fra lovtekstens `jf. §`-refs
4. Først DERETTER: legg til nye dokumenter (DiBK-veiledere etc.)

**4. Mål → forbedring → mål → behold/forkast.**

Hver endring er et eksperiment. Eckhoff-metrikker før og etter.
Regresjon = revert. Ingen endring uten målbar forbedring.

**5. Kapittel-ekspansjon er implementert — bruk den.**

`HypergraphQuery.__init__` ekspanderer nå kap:-kanter til §§
automatisk. Forarbeider koblet til `kap:pbl-19` indekseres under
alle `lov:pbl§19-*` seksjoner. Dette betyr at `resolve_question_ranked`
finner forarbeider og lovtekster som den aldri fant før.

**6. Eckhoff-diversitetsinjeksjon sikrer bredde.**

`resolve_question_broad()` injiserer automatisk den beste
court_ruling, preparatory_work og circular hvis de mangler fra
topp-resultatene. Dette sikrer at brukeren alltid ser kilder fra
flere rettskildenivåer — selv om tolkningsuttalelser dominerer
RRF-scoren.

### Mål

Optimalisere systemets evne til å finne **juridisk presise kilder fra
alle relevante nivåer i Eckhoffs rettskildehierarki** for et gitt
brukerspørsmål. Grafen er søkeruten: spørsmål → §§ → rettskilder
fra alle 7 nivåer → helhetlig juridisk vurdering.

### Eckhoffs rettskildehierarki og nåværende dekning

| Rettskildenivå | Type i graf | Antall | §§ dekket | Status |
|----------------|-------------|--------|-----------|--------|
| 1. Lovtekst | `law_text` | 36 | 265 | ✅ Via kapittel-ekspansjon |
| 2. Forskriftstekst | `regulation_text` | 36 | 275 | ✅ Via kapittel-ekspansjon |
| 3. Forarbeider | `preparatory_work` | 7 | 109 | ✅ Via kapittel-ekspansjon |
| 4. Rettspraksis | `court_ruling` | 54 | 58 | **Moderat — trenger §-parsing** |
| 5. Forvaltningspraksis (KDD) | `document` | 368 | ~264 | Bra |
| 6. Sivilombudet | `document` | 259 | ~180 | Bra |
| 7. Rundskriv/veiledere | `circular` | 21 | 111 | Moderat |
| 8. DiBK-veiledere | `guidance` | 580 | 277 | ✅ Integrert (sesjon 32) |

### Eckhoff-dybde per § (oppdatert 2026-03-21)

| Kildetyper | §§ | Andel | Visuell |
|------------|-----|-------|---------|
| 7 | 12 | 2.5% | ████ |
| 6 | 40 | 8.2% | █████████████ |
| 5 | 68 | 13.9% | ██████████████████████ |
| 4 | 75 | 15.3% | █████████████████████████ |
| 3 | 104 | 21.3% | ██████████████████████████████████ |
| 2 | 124 | 25.4% | ████████████████████████████████████████ |
| 1 | 59 | 12.1% | ███████████████████ |
| 0 | 7 | 1.4% | ██ |

**Snitt dybde: 3.24** | **§§ med ≥3 kildetyper: 299 (61.1%)**

### Søkeruten: Spørsmål → Eckhoff-vurdering (implementert)

```
Brukerspørsmål
    │
    ▼
Trestegs-klassifisering (classifier.py)
    │  → Identifiserer 1-8 relevante §§
    ▼
Kapittel-ekspansjon + hierarki-ekspansjon
    │  → kap:pbl-19 → lov:pbl§19-1, §19-2, §19-3, §19-4
    │  → lov:pbl§20-5 → lov:sak10§4-1 (implementerende forskrift)
    ▼
6 parallelle søkekanaler → RRF-fusjon
    │  → Graf, paragraf-refs, keywords, BM25, embedding, hierarki
    ▼
Eckhoff source-type boost
    │  → court_ruling +0.010, preparatory_work +0.008, circular +0.004
    ▼
Eckhoff-diversitetsinjeksjon
    │  → Mangler court_ruling i topp? → injiser beste kandidat
    │  → Mangler preparatory_work? → injiser beste kandidat
    │  → Mangler circular? → injiser beste kandidat
    ▼
15 resultater med 4+ Eckhoff-kildetyper
```

### Implementasjonsstatus

| Komponent | Status | Effekt |
|-----------|--------|--------|
| Eckhoff-metrikker (`diagnose_hypergraph.py`) | ✅ | Dybde: 3.24 (opp fra 2.29) |
| Kapittel→§ indeks-ekspansjon | ✅ | Forarbeider: 12→109 §§ |
| Eckhoff source-type boost i RRF | ✅ | HR/forarbeid rangeres høyere |
| Eckhoff-diversitetsinjeksjon | ✅ | 4 kildetyper i alle Eckhoff-cases |
| Tekst-søk utvidet til alle kildetyper | ✅ | court_ruling/circular søkbare |
| 5 Eckhoff-benchmark-cases | ✅ | Recall 65%→80% (Ranked) |
| Autonom eksperiment-loop | ❌ | Fase 2-3 |
| Forarbeider §-parsing | ❌ | Fase 2 |
| Rettspraksis §-parsing | ❌ | Fase 2 |

### Sikkerhetsregler

- **Loopen kjører ALDRI på hovedbranchen.** All eksperimentering skjer
  på `autoresearch/hypergraf-forbedring` eller sub-brancher.
- **Hovedbranchen berøres kun via PR** etter manuell gjennomgang.
- **`git reset --hard`** brukes kun på autoresearch-branchen ved regresjon.
- **`hypergraph.json`** på hovedbranchen forblir uberørt under eksperimentering.

### Primærmetrikker

| Metrikk | Beskrivelse | Baseline | Mål |
|---------|-------------|----------|-----|
| `eckhoff_avg_depth` | Snitt Eckhoff-dybde per § | **3.24** | ≥4.0 |
| `eckhoff_3plus_pct` | §§ med ≥3 kildetyper | **61.1%** | ≥75% |
| `broad_recall` | Recall over 20 benchmark-cases | **72%** | ≥80% |
| `ranked_recall` | Recall (ranked metode) | **80%** | ≥85% |

### Sekundærmetrikker

| Metrikk | Kilde | Baseline |
|---------|-------|----------|
| `orphaned_pct` | `diagnose_hypergraph.py` | 8.0% |
| `rettspraksis_coverage` | §§ med court_ruling | 82 |

### Forbedringstargets (prioritert etter Eckhoff-impact)

1. ~~**Eckhoff-vekting i rangering**~~ ✅ Implementert
2. ~~**Kapittel→§ ekspansjon**~~ ✅ Implementert (forarbeider 12→109 §§)
3. ~~**Eckhoff-diversitetsinjeksjon**~~ ✅ Implementert
4. **Rettspraksis §-parsing** — 54 dommer har tekst med §-refs.
   Parse disse og opprett direkte hyperkanter. Mål: 58→100+ §§.
5. **Forarbeider §-parsing** — 7 proposisjoner nevner spesifikke §§
   i teksten. Parse og opprett §-nivå kanter (utover kapittel).
6. ~~**DiBK-veiledere**~~ ✅ Integrert — 580 guidance-noder, 277 §§ dekket.
7. **Kryssreferanse-resolusjon** — 55 §§ med refs i tekst uten kanter

### Eksperiment-syklus

```
1. python scripts/diagnose_hypergraph.py → Eckhoff-baseline
2. Gjør én forbedring (kobling, ny kilde, §-parsing)
3. python scripts/build_hypergraph.py → rebuild
4. python scripts/diagnose_hypergraph.py → mål delta
5. python tests/test_broad_benchmark.py → recall/rangering
6. Forbedret? → git commit. Regresjon? → git reset --hard
7. Logg til results.jsonl → gjenta
```

### Gold doc kvalitetskontroll

Etter endringer i gold docs-lister, kjør **alltid** denne prosedyren:

```bash
# 1. Verifiser at alle gold docs finnes i grafen
python scripts/verify_gold_docs.py

# 2. Kjør benchmark for å se recall-effekt
python tests/test_broad_benchmark.py

# 3. Kjør refresh-rapport for å se nye kandidater
python scripts/refresh_gold_docs.py --summary
```

**Ved tillegg av nye gold docs — manuell vurdering er OBLIGATORISK:**

1. Les dokumentets faktiske innhold (tittel + snippet), ikke bare score
2. Sjekk: adresserer dokumentet **direkte** spørsmålets juridiske tema?
3. Sjekk: finnes noden faktisk i grafen? (`hq.node(doc_id)`)
4. **ALDRI** legg til basert kun på søkescore — høy score ≠ relevant innhold
5. **ALDRI** legg til lovkommentar-noder (`lovkommentar:*`) — de eksisterer
   ikke som graf-noder og vil bare trekke ned recall

Verktøy for manuell vurdering:
- `scripts/review_gold_docs.py --case N` — viser spørsmål + kandidater med innhold
- `scripts/verify_gold_docs.py` — verifiserer at alle gold docs eksisterer
- `scripts/apply_manual_gold_updates.py` — mal for dokumenterte beslutninger

### Nøkkelfiler

| Fil | Rolle |
|-----|-------|
| `scripts/experiment_loop.py` | Hovedscript for autonom loop (GJENSTÅR) |
| `results.jsonl` | Append-only eksperimentlogg (ucommittet) |
| `scripts/diagnose_hypergraph.py` | Metrikk-måling med Eckhoff-dekning |
| `scripts/build_hypergraph.py` | Rebuild hypergraf |
| `tests/test_broad_benchmark.py` | Søke-benchmark (103 cases) |
| `scripts/verify_gold_docs.py` | Verifiser gold docs gyldighet |
| `scripts/review_gold_docs.py` | Manuell vurdering av gold doc-kandidater |
| `byggsok/hypergraph_query.py` | Query-engine med Eckhoff-vekting + injeksjon |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Minervapra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
