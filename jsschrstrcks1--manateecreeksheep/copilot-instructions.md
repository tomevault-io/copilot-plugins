## manateecreeksheep

> This repository manages the flock records, breeding program, and health tracking for a sheep operation at Manatee Creek in Florida. It serves three purposes:

# Manatee Creek Sheep — Project Guidelines

## Purpose

This repository manages the flock records, breeding program, and health tracking for a sheep operation at Manatee Creek in Florida. It serves three purposes:

1. **Flock management** — maintaining accurate records of every animal: identity, pedigree, breed composition, health, pen assignments, and lambing
2. **Breeding program** — tracking genetics across generations to make informed breeding decisions, maximize hybrid vigor, and manage inbreeding avoidance
3. **Image archive** — preserving photographs (spiral notebook notes, handwritten records, sheep photos) as primary source material

This is a family operation. Accuracy matters because real animals depend on correct records. Every data point affects breeding decisions, health interventions, and flock management.

---

## Theological Foundation

> *"The LORD is my shepherd; I shall not want."* — Psalm 23:1 (ESV)

**Soli Deo Gloria** — All work on this project is offered as a gift to God. We tend these sheep as stewards, not owners.

---

## Data Sources — Priority Order

The flock data comes from multiple sources. When sources conflict, use this priority:

| Priority | Source | Description |
|----------|--------|-------------|
| **1 (HIGHEST)** | Spiral notebook images (PNG files) | Mom's phone app notes — most current, most accurate reflection of what's on the ground |
| **2** | `flock_record_v2.xlsx` | Structured spreadsheet with pen rosters, lambing records, cross-references |
| **3** | `data.csv` | Historical records with breed percentages, weights, DOBs |
| **4** | Google Sheet | Breed composition calculations for potential offspring by pen |
| **5** | `Sheep_Breeding_DB_CURRENT_COPY.xlsx` | Mating plans, ram eligibility, breeding rules |

**Important aliases:**
- Mom writes "Amure" — this is **Azure**
- "Rock" = "Jerkface" = Awassi ram
- "Bt" = Broken Tail
- "Bsoe" / "Bsoed" are separate sheep (tags 31/32, switched)
- "GG" = Gigi
- Mc11 = Charlie's ram = tag 12
- Mc12 = 036 = Serendipity's baby ewe
- Mc01 = Little Daisy's baby (baby's baby) = tag 35
- NoriSon = ram in pen 5, tag 54

---

## Image Processing

### Critical Rule: Images Must Be Processed Before Reading

All original images (JPG and PNG) are 4032x3024px or 1320x2868px — both exceed Claude's 2000px API limit.

**Always use processed versions from `data/processed/`**

```bash
# Check image status
python3 scripts/process_images.py --status

# Process all oversized images
python3 scripts/process_images.py

# Process a single image
python3 scripts/process_images.py --file IMG_8560.JPG
```

### Image Categories

| Range | Type | Content |
|-------|------|---------|
| IMG_8560–8615 | JPG | Handwritten notebook pages (sheep check records with photos) |
| IMG_8616–8643 | PNG | Phone app notes (treatments, measurements, pen assignments, health notes) |

### Image Retention Policy

**NEVER delete any image.** Every image is a primary source document. Original handwritten records and photos of the sheep are irreplaceable.

---

## Database Schema

The flock database lives at `data/flock_database.json`. It contains:

- **sheep[]** — Every animal past and present with pedigree, breed composition, health, pen, status
- **pens{}** — Current pen assignments with breeding groups
- **lambing_records_2026[]** — 2026 lambing season records
- **breed_reference{}** — Classification of breeds as hair, wool, or dual-purpose

### Key Fields Per Sheep

| Field | Description | Required |
|-------|-------------|----------|
| `id` | Unique kebab-case identifier | Yes |
| `name` | Display name | Yes |
| `tag` | Ear tag number (if any) | No |
| `sex` | ram, ewe, ram_lamb, ewe_lamb, wether, unknown | Yes |
| `breed_composition` | Percentages of each breed | When known |
| `sire_id` / `dam_id` | References to parent sheep IDs | When known |
| `status` | alive, deceased, sold, unknown | Yes |
| `pen` | Current pen assignment | When known |
| `health` | FAMACHA scores, treatments, weak resistance flag | Ongoing |
| `confidence` | high, medium, low — how certain we are of the data | Yes |

### Breed Composition

Track percentages for all breeds present. Common breeds in this flock:

**Hair breeds:** Katahdin, Dorper, White Dorper, St Croix, Barbados Blackbelly (BBB), American Blackbelly (ABB), Wiltshire Horn
**Wool breeds:** Suffolk, Hampshire, Cotswold, Tunis, Gulf Coast Native (GCN)
**Dual-purpose:** St Augustine, Cracker, Awassi, East Friesian
**Other:** Jacob, Babydoll, Karakul

---

## Breeding Program Principles

### Long-Term Vision

The long-term goal is to produce sheep that are **MORE parasite resistant** while remaining **meaty and milky**. The flock draws from **22 breeds** — sheep whose former owners claimed no parasite issues. This has proved more or less true in most cases.

### Key Genetics Observations
- **Most parasite resistant:** Kelsier (Katahdin) — the gold standard in this flock
- **Most milky:** Awassi and Awassi crosses
- **Meatiest:** Dorper-Awassi cross
- **Best all-around:** Katahdin foundation with strategic crosses

### Selection Priorities
1. **Parasite resistance** — FAMACHA scoring drives culling and breeding decisions. This is the #1 priority.
2. **Meat quality** — Maintain body condition and growth rates (Dorper influence)
3. **Milk production** — Preserve Awassi genetics for dairy potential
4. **Hair coat selection** — Moving toward hair sheep (less maintenance, better for Florida heat)
5. **Hybrid vigor** — Strategic crossbreeding across 22 breeds for diverse genetic resistance
6. **Inbreeding avoidance** — Track all pedigrees to prevent close matings

### Breeding Rules (from Sheep_Breeding_DB)
- R1: Exclude placeholder/unknown "twin rams" from recommendations
- R2: Only recommend adult rams (>=9 months) with known DOB
- R3: Geriatric safety: If ewe age >= 6 years, exclude Ram 00110 (Orange Tag) — reduce dystocia risk
- R4: Only recommend rams marked "Eligible"

### FAMACHA Scoring
- Score 1: Red (healthy) — no treatment needed
- Score 2: Red-pink — no treatment needed
- Score 3: Pink — borderline, monitor closely
- Score 4: Pink-white — treat
- Score 5: White (severe anemia) — treat immediately, consider culling

### Weak Resistance List (Feb 14, 2026)
These animals have shown poor parasite resistance and should be watched closely or considered for culling:
Shaggy (deceased), GG, Azure, Butter Ball (deceased), Rocky, Skitters (deceased), Dorper 23 & 25, W136 (deceased), Circle Tail, W140, FM1, Samson (deceased), Unnamed (deceased), Baby, Bella, FM

---

## Health Management

### Common Treatments
- **Corid** — for coccidia (5-day protocol)
- **Ivermectin** — dewormer
- **Fenbendazole** — dewormer (often combined with ivermectin)
- **Iron supplement** — for anemia
- **Vitamin B injection** — supportive care
- **Pen-C (Penicillin)** — for respiratory infections

### Treatment Records
All treatments should be logged in the database with:
- Date
- Animal ID
- Treatment given
- FAMACHA score at time of treatment
- Pen location
- Notes

---

## Pen Structure (as of Feb 2026)

| Pen | Ram | Notes |
|-----|-----|-------|
| Pen 1 | Kaladin | With Eclipse, Merrie, Abg, Fm |
| Pen 2 | Sir Loin | With Azure, S2, Lara, Bambii, Unnamed, Pebbles |
| Pen 3 | Sam | With Baby, Baby momma, Zara, Half tail, New big girl 2 |
| Pen 4 | Samson | With Elsie, Nori, Trouble, Bsoe, Bsoed, Banana |
| Pen 5 | Rocky/NoriSon | With Amber 24, Broken tail, Little daisy; NoriSon tag 54 |
| Pen 6 | No ram | Shaggy, Serendipity, S1, Fm1, Fox tail, Circle tail |
| Goose Pen | — | Auction lambs (09, 50, 06, L19, others) |

---

## Validation

```bash
# Validate the flock database
python3 scripts/validate_flock.py

# Check for orphan references (sire/dam IDs that don't exist)
python3 scripts/validate_flock.py --check-references

# Verify image references
python3 scripts/validate_flock.py --check-images
```

---

## Non-Negotiable Rules

1. **NEVER delete images** — they are primary source documents
2. **NEVER invent data** — if something is unclear, mark it `[UNCLEAR]` and note confidence as "low"
3. **Spiral notebook is authoritative** — when sources conflict, the notebook wins
4. **"Amure" = Azure** — Mom's spelling; always normalize to "Azure" in the database
5. **Track deceased animals** — they are still valuable for pedigree records
6. **Always process images before reading** — never read originals >2000px directly
7. **Commit after each logical unit of work** — don't batch unrelated changes
8. **Verify before reporting** — check your work against the images before saying "done"

---

## Working Notes

- Most recent lambing season: February 2026 (14 dams lambed, ~19+ lambs born as of 2-13-26)
- Vaccination day: May 12, 2025 (49 vaccinated, 48 parasite treatment)
- Next steps: Continue lambing records, update pen assignments as lambs are weaned, plan next breeding season

---

## Integrity Rule: Careful, Not Clever

> **Active skill: `.claude/skills/careful-not-clever/SKILL.md`**
> **Readable reference: [careful.md](careful.md)**

The governing principle: everything we do is for the glory of God and with integrity. We don't guess — we verify. We don't assume — we check. We don't cut corners — we serve the flock.

**Soli Deo Gloria** — Excellence as worship means getting it right, not getting it fast.

---

## Multi-LLM Integration

This repository has access to the multi-LLM orchestrator system. External models (GPT, Gemini, Grok) serve as **consultants only** — Claude remains lead author and decision-maker.

### Available Skills

| Skill | Usage | Purpose |
|-------|-------|---------|
| `/consult` | `/consult gpt plan "breeding plan for spring lambing"` | Quick single-model second opinion |
| `/orchestrate` | `/orchestrate sheep "evaluate fall breeding pairs"` | Full multi-model pipeline |
| Cognitive Memory | Automatic on session start | Cross-session knowledge persistence |

### Mode: `sheep`
- **Lead:** GPT (planning), Claude (safety/validation)
- **Pipeline:** Plan (GPT) → Context (Gemini) → Challenge (Grok) → Validate (Claude) → Finalize (GPT)
- **Memory scope:** `/sheep`
- **Orchestrator:** `/home/user/ken/orchestrator/`

### First-Time Setup (Per Session)

Before first use of `/consult` or `/orchestrate` in a session, install dependencies:

```bash
bash /home/user/ken/orchestrator/bootstrap-env.sh 2>/dev/null
pip3 install -q -r /home/user/ken/orchestrator/requirements.txt
```

This is silent when already installed. If `/consult` or `/orchestrate` fails with `ModuleNotFoundError`, run this command first.

### Context Boundaries
- **SEND:** Anonymized flock data, breeding objectives, trait scores, health summaries
- **NEVER SEND:** Financial records, location details beyond "Florida"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jsschrstrcks1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
