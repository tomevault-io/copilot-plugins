## santanna-pisa-monitor-skill

> Monitor job postings, admissions, courses, and research opportunities from Scuola Superiore Sant'Anna di Pisa


# Sant'Anna Pisa - Job and Course Monitor

Monitors the Scuola Superiore Sant'Anna di Pisa website for job postings, admissions, courses, and educational/employment opportunities.

**Website:** https://www.santannapisa.it/it
**No API keys or dependencies required** — uses `browser_navigate` and `browser_snapshot`

## Key Pages

### JOBS / SELECTIONS
| Page | URL | Content |
|------|-----|---------|
| Competitions hub | `/it/ateneo/concorsi-selezioni-e-gare` | Central hub with all categories (jobs + education) |
| Active selections | `/it/cerca-selezioni?status=current` | Currently open job postings |
| Expiring selections | `/it/cerca-selezioni?status=expiring` | Job postings closing soon |
| Selection archive | `/it/cerca-selezioni?status=expired` | Closed/expired postings |

### COURSES / EDUCATION
| Page | URL | Content |
|------|-----|---------|
| Enter Sant'Anna | `/it/entrare-al-sant-anna` | **Main hub** — admissions deadlines for all levels |
| Bachelor admission | `/it/concorso-di-primo-livello-e-ciclo-unico` | Entrance exam info and deadlines for Bachelor (3-yr) |
| Master admission | `/it/concorso-di-secondo-livello` | Entrance exam info and deadlines for Master (2-yr) |
| PhD call | `/it/formazione/bando-phd` | **Current PhD call** with positions and deadlines |
| PhD overview | `/it/formazione/phd` | List of all PhD programs |
| Master I level | `/it/master-i-livello` | 1st level Master programs list |
| Master II level | `/it/master-ii-livello` | 2nd level Master programs list |
| Seasonal School | `/it/formazione/seasonal-school` | Short intensive courses with dates |
| How to apply (Seasonal) | `/it/formazione/come-accedere-seasonal-school` | Seasonal School application info |

## Job Selection Categories (URL Filters)

Append `?categories[0]=CATEGORY_NAME` to `/it/cerca-selezioni`:

| Category | Filter URL | Description |
|----------|-----------|-------------|
| Teaching/research staff | `personale_docente_e_ricercatore` | Professors and researchers |
| Fixed-term researchers | `selezioni_ricercatori_a_temp_determinato` | RTD-A, RTD-B positions |
| Research contracts | `contratto_di_ricerca` | Research grants/contracts |
| Post-doc | `incarichi_post_doc` | Post-doctoral positions |
| Research positions (direct) | `incarichi_di_ricerca_selezione` | Direct selection |
| Expression of interest | `incarichi_di_ricerca_interesse` | Open call |
| Research grants archive | `assegni_di_ricerca` | Archived grants |
| Teaching | `incarichi_di_insegnamento` | Teaching assignments |
| Technical-admin staff | `selezioni_personale_tecnico_amm` | Technical/administrative staff |
| Technologist | `selezioni_personale_tecnologo` | Technologist roles |
| External assignments | `incarichi_esterni` | External collaborations |

## How to Monitor (Complete Workflow)

### SECTION A: JOBS

From the snapshot of `/it/cerca-selezioni`, each posting has this structure:
```
- article:
    - text: Data pubblicazione bando DD.MM.YYYY
    - text: ID Bando: NNN/YYYY
    - heading "Title" [ref=eXX] [level=3]:
      - link "Title" [ref=eXX]: - /url: "/it/assegni-di-ricerca-e-selezioni-incarichi-esterni/..."
    - text: Termine presentazione domanda DD.MM.YYYY
```

### SECTION B: COURSES / EDUCATION — DEADLINES

#### Bachelor/Master Admission (from /it/entrare-al-sant-anna)
```
- heading "Bando del Concorso di Ammissione YYYY/YYYY"
- "Termine iscrizione domanda Concorso I livello:" → DD month YYYY, ore HH:MM
- "Termine iscrizione domanda Concorso II livello:" → DD month YYYY, ore HH:MM
- PDF: https://www.santannapisa.it/sites/default/files/YYYY-MM/Bando%20unico%20Concorso%20di%20Ammissione%20I%20e%20II%20livello%20YYYY-YY.pdf
```

#### PhD Call (from /it/formazione/bando-phd)
```
- heading "Bando PhD YYYY/YYYY"
- "NN positions"
- General deadline: "DD month YYYY, ore HH:MM"
- PhD Economics deadline: "DD month YYYY, ore HH:MM" (earlier)
- PDF: https://www.santannapisa.it/sites/default/files/.../Bando%20concorso%20PhD%20...pdf
- Online application: https://sssup.esse3.cineca.it/Home.do
```

#### Seasonal School (from /it/formazione/seasonal-school)

The page has ~36 courses displayed in a 3-column card layout. **Scroll down 3-4 times** to capture all courses.

```
- heading "Programma YYYY-YYYY"
- For each course (card):
    - heading level 3: Course name
    - link: /it/seasonalschool/course-name
    - paragraph: dates (e.g. "March 23rd - 27th, 2026")
- Previous program archives: 24-25, 23-24, 22-23, 21-22
```

For application deadlines: navigate `/it/formazione/come-accedere-seasonal-school`
- Winter courses: deadlines typically ~2 months before
- Spring courses: deadlines ~1-2 months before  
- Summer courses: deadlines ~1 month before

#### University Masters (from /it/master-i-livello and /it/master-ii-livello)
Each Master has its own page with:
- Master title
- Application deadline
- PDF call link

## Complete Monitoring (Cronjob Workflow)

For a full check, navigate in order:
1. `/it/cerca-selezioni?status=expiring` — job postings closing soon
2. `/it/cerca-selezioni?status=current` — active job postings
3. `/it/entrare-al-sant-anna` — Bachelor/Master admission deadlines
4. `/it/formazione/bando-phd` — PhD call status
5. `/it/formazione/seasonal-school` — Seasonal School courses (scroll 3-4x to capture all)
6. `/it/formazione/come-accedere-seasonal-school` — Seasonal School application info
7. `/it/master-i-livello` — 1st level Master programs
8. `/it/master-ii-livello` — 2nd level Master programs

## URL Construction
Relative links (e.g. `/it/...`) should be prefixed with `https://www.santannapisa.it` to form full URLs.

## Notes and Pitfalls
- Drupal-based site, server-side rendering, no RSS feeds available
- Cookie banner present on most pages — ignore it
- PhD call URL changes yearly but `/it/formazione/bando-phd` always points to the current one
- Bachelor/Master admission PDF URLs change annually
- Always compare with previous check to detect new postings or changes
- Individual pages for future Summer School courses (April-July) may not be published yet — check periodically

---
> Source: [FabioBaroni/santanna-pisa-monitor-skill](https://github.com/FabioBaroni/santanna-pisa-monitor-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
