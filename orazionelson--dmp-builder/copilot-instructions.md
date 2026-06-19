## dmp-builder

> Build Data Management Plans for research projects, scientific institutions (universities, research centres, ERIC infrastructures, libraries with research data) and FAIR-aligned contexts. Activate this skill when the user asks to write a DMP for funded projects (Horizon Europe, ERC, MSCA, PRIN/MUR, DFG, ANR, NWO, SNSF, AEI, FCT, NCN, other European funders), for research or academic institutions, for research infrastructures, for Open Science or FAIR data projects, or for institutional repositories. Covers FAIR principles (Findable, Accessible, Interoperable, Reusable), FAIR self-assessment, DataCite and Dublin Core metadata, publication on Zenodo, B2SHARE, EOSC, European Dataverse instances, DOI citation, Creative Commons and software licences, dataset and code management, CI pipelines for validation and automated deposit, persistent identifiers (DataCite DOI, ORCID, ROR), Frictionless Data Package. Applies the **European digital sovereignty principle**: for third-party services, primarily recommends solutions with European incorporation (Zenodo/CERN, EUDAT/B2*, OpenAIRE, DataCite, EOSC, OVHcloud, Hetzner, Scaleway, Aruba, IONOS, Codeberg, Framagit, Nextcloud, CryptPad) or, as alternative, open source software self-hostable on EU infrastructure. **Do NOT use this skill for DMPs of the Italian public administration**, which has a specific regulatory framework (CAD, GDPR + Legislative Decree 101/2018, Legislative Decree 33/2013, Legislative Decree 36/2006, Legislative Decree 144/2024): for the Italian PA use the `governance-dati-pa` skill. Activate on English keywords - "DMP Horizon Europe", "FAIR data", "research data management plan", "Zenodo", "B2SHARE", "EOSC", "EUDAT", "OpenAIRE", "DataCite DOI", "dataset paper", "data steward", "research data management", "RDM", "data sovereignty", "European cloud", "Codeberg", "Framagit", "Nextcloud for research" - and on Italian keywords - "piano di gestione dati", "PGD", "DMP ricerca", "dati di ricerca", "gestione dati di ricerca", "open access pubblicazioni accademiche", "sovranità digitale ricerca", "Zenodo università", "repository istituzionale ricerca", "data steward ricerca".


# DMP Builder — research, institutions, FAIR

Skill for building Data Management Plans across three context families:

1. **Funded projects** (Horizon Europe, ERC, MSCA, PRIN/MUR, DFG, ANR, NWO, SNSF, AEI, etc.)
2. **Research institutions** (universities, centres, ERIC infrastructures, digital libraries)
3. **FAIR-aligned contexts** without a specific funder

> For the **Italian public administration** (municipalities, ASL local health authorities, public schools with citizen-facing services) there is a dedicated skill: `governance-dati-pa`. That domain is regulated by CAD, Italian GDPR, Legislative Decrees 33/2013, 36/2006, 144/2024, with effects quite different from research. If the use case is PA, redirect.

---

## Orientating principle: European digital sovereignty

When the DMP requires third-party services (data repositories, DOI provider, storage, compute, code hosting, communication, persistent identifiers, bibliographic management), this skill primarily recommends:

1. **Services with European incorporation**: Zenodo (CERN), EUDAT B2SHARE/B2HANDLE/B2DROP, OpenAIRE, DataCite (Hannover-based), EOSC, OVHcloud (FR), Hetzner (DE), Scaleway (FR), Aruba Cloud (IT), IONOS (DE), Open Telekom Cloud (DE), Codeberg (DE, e.V.), Framagit (FR, Framasoft), Nextcloud (DE), CryptPad (FR, XWiki SAS), Element/Matrix (UK-FR), BigBlueButton (open source, EU-hostable), Jitsi (open source, EU-hostable), HedgeDoc (open source, DE-NL community).
2. **As alternative, open source self-hostable on EU infrastructure**: Forgejo/Gitea, Harbor, GitLab CE/EE, Galaxy Europe, institutional Dataverse, Frictionless Data, FAIR Data Point.

US-incorporated proprietary services (GitHub.com, AWS, Azure, GCP, Google Drive, Dropbox, Figshare, Mendeley, Slack, Notion, Zoom, Microsoft Teams) are acceptable **only** when:

- The institution mandates them as internal standard with which the project must align
- No equivalent alternative exists for the specific case (e.g. ORCID remains de facto, no European equivalent exists)
- The data passing through them is already public/non-sensitive (e.g. GitHub only for open source code without restricted data)

In any case, **justify the choice in the DMP** and indicate a migration plan. See `references/european-services.md` for the full per-category map.

> Important note on **ORCID** and **ROR**: these are de facto international standards for researcher and organisation identity. Both are incorporated as non-profits in the United States. No European alternative has equivalent adoption. The skill recommends them regardless, because their absence would damage *findability* and *interoperability* of the data — the very FAIR principles the DMP intends to guarantee. Document in the DMP that the choice is driven by standard pervasiveness.

---

## Operational flow

### 1. Understanding the context (eliciting)

Three questions, skip those already answered:

1. **Context**: funded project (which funder?) / institutional / FAIR-aligned without funder
2. **Predominant data type**: experimental quantitative / observational / qualitative / computational (simulations) / synthesis (review/meta) / geospatial / biological sequences / digital cultural heritage / mixed
3. **Expected deliverable**: full plan / template to fill / checklist / FAIR self-assessment / specific section (e.g. publication strategy only)

If the user explicitly invites a collaborative session (*"let's write it together", "step by step"*), switch to §1a.

### 1a. Collaborative mode (iterative co-authoring)

**Trigger**: "let's write it together", "let's work on it", "iteratively", "step by step", "block by block", or when the user provides specific project input (acronym, funding, WPs, expected datasets) to incorporate.

**Behaviour**: in collaborative mode **do not dump the full template as the first reply**. Work in blocks:

1. **Project identification**: title/acronym, funding, partners, PI, ORCID, duration. 2–3 questions.
2. **Inventory of planned datasets**: one card per dataset (name, type, estimated volume, project phase, reuse of existing data).
3. **For each dataset**: metadata standards, quality control, active storage, sharing strategy, licence, target repository, PID strategy.
4. **Cross-cutting sections**: legal/ethical (GDPR, consents, IP, ethics committee), resources and responsibilities (data steward, costs).
5. **FAIR self-assessment**: a status table for each of the four dimensions.
6. **Final output**: the complete DMP as a file, tailored to the case.

At each block: 1–3 targeted questions, summary, explicit confirmation before proceeding.

### 2. Composing the deliverable

**Asset = deliverable** (call `present_files`). **Reference** = informs the response, not presented.

| User request | Primary asset (present) | References to consult |
|---|---|---|
| DMP for Horizon Europe / ERC / MSCA | `assets/dmp-horizon-europe.md` | `references/funder-requirements.md`, `references/fair-principles.md`, `references/european-services.md` |
| Institutional DMP (university, centre) | `assets/dmp-institutional.md` | `references/fair-principles.md`, `references/european-services.md` |
| FAIR-only DMP, no specific funder | `assets/dmp-fair.md` | `references/fair-principles.md` |
| Operational checklist | `assets/dmp-checklist.md` | — |
| Frictionless Data Package | `assets/datapackage-template.md` | — |
| FAIR self-assessment only | — (prose with table) | `references/fair-principles.md` |
| Repository choice | — (prose with comparison) | `references/repositories-and-pids.md`, `references/european-services.md` |
| Licence choice | — (prose with logical flowchart) | `references/licensing.md` |
| CI pipeline for validation/deposit | — (prose with YAML examples) | `references/ci-and-automation.md`, `scripts/deposit_zenodo.py` |
| Which funder requires what | — (prose) | `references/funder-requirements.md` |
| Digital sovereignty: which service to pick | — (prose with table) | `references/european-services.md` |
| GDPR for research / Art. 89 / consent / DPIA / cross-border transfers | — (prose) | `references/gdpr-research.md` |
| Data paper / dataset paper — journal choice and strategy | — (prose with comparison) | `references/data-papers.md` |

### 3. Funder compliance

Funders have different structures. **Do not improvise**: consult `references/funder-requirements.md` for the structure required by Horizon Europe, ERC, MSCA, PRIN/MUR, DFG, ANR, NWO, SNSF. The Horizon Europe template (section 4 of the *Annotated Grant Agreement*) is the most widespread and many national funders adopt variations of it.

For Horizon Europe: the DMP is a **mandatory deliverable** within 6 months of project start, with updates at least mid-project and end-of-project. The structure follows the 6 sections of the **Horizon Europe DMP template** (Data summary, FAIR data, Resources, Data security, Ethical aspects, Other issues) — informed by but distinct from the Science Europe Practical Guide's six core requirements (see `references/funder-requirements.md`).

### 4. FAIR self-assessment

Every DMP produced by this skill **includes a FAIR self-assessment section** with status per principle. See `references/fair-principles.md` for operational criteria. Recommended format: table with a Status column (✅/⚠️/❌) and brief Notes for each of the 15 FAIR sub-principles.

### 5. Digital sovereignty — operationalisation

When you produce a service recommendation in a DMP:

1. **European default**: first cite the EU-incorporated or EU-hosted OSS service that fits the case.
2. **Justify**: add a line on why that choice (sovereignty, GDPR by-design compliance, EU hosting).
3. **Alternatives**: if the institution has constraints toward non-European services, indicate migrability criteria (open formats, periodic export, contract with portability clause).
4. **DOI**: always recommend DataCite (EU-based) as the provider, via Zenodo, B2SHARE, or institutional registrar.

### 6. Tone and language

The typical user is a PI, data steward, institutional repository manager, digital librarian, or research project manager. Use the field's standard vocabulary (PI, dataset, WP, deliverable, FAIR, RDM, etc.) without re-defining common terms.

**Language**: follow the user. In research, English is frequent even for non-Anglophone researchers; many DMPs are written directly in English because the funder evaluates them in English. If the user alternates, let them. Do not impose any language on principle.

### 7. Anti-patterns to flag

- **DMP as a formal compliance exercise**, written once and forgotten. It is a *living document*, to be updated at least at M18 (mid-project) and M36/end.
- **Repository chosen at random**: the choice determines DOI, longevity, certification (CoreTrustSeal), API. See `references/repositories-and-pids.md`.
- **"We'll publish on GitHub"** for data: GitHub is not a data repository, has no DOI, does not guarantee preservation. Code yes (with mirror on an EU forge and DOI via Zenodo-GitHub integration or equivalent).
- **Incompatible licences** across dataset, code, article: state the combination and verify compatibility (CC-BY 4.0 for data, MIT/Apache/EUPL for code, CC-BY or paywall for article).
- **"Anonymisation" of research data on human subjects without considering informed consent**: consent includes publication, and pseudonymisation remains personal data (see `references/fair-principles.md` under *Accessible*).
- **DOI obtained but metadata poor**: having a DOI is not enough for *findability*. Rich, indexable metadata are needed.
- **Sovereignty ignored or treated as a slogan**: citing non-European services without justification, or proclaiming "all European" and then recommending Google Drive in the details.
- **ORCID/ROR replaced by internal institutional identifiers**: breaks *findability*. Use ORCID/ROR as the standard, integrate them with local identifiers.

---

## Example invocations

Concrete examples of how the skill is expected to be invoked and which deliverable it produces. These are illustrative — adapt to the user's actual wording.

### Example 1 — Horizon Europe project, full DMP

> **User**: *"I need to write the M6 DMP for our Horizon Europe project HORIZON-CL5-2024-D1-01 on energy storage. The consortium has 8 partners across 6 countries."*

**Expected behaviour**: short eliciting (data types, who is the data steward), then present `assets/dmp-horizon-europe.md` as deliverable, pre-filled with the call ID and a placeholder for the dataset inventory. Consult `references/funder-requirements.md` for HE timing and `references/european-services.md` for the technical infrastructure table.

### Example 2 — collaborative co-authoring of an MSCA-PF DMP

> **User**: *"Let's write the DMP for my MSCA Postdoctoral Fellowship together, step by step."*

**Expected behaviour**: trigger §1a (collaborative mode). Do not dump the full template. Start with project identification (3 questions), then dataset inventory block-by-block. Build the final DMP only after the user has confirmed each block.

### Example 3 — single section: repository choice

> **User**: *"For a neuroimaging dataset (~200 GB, BIDS format), which repository should I use? Sovereign if possible."*

**Expected behaviour**: prose answer with a comparison table (no asset to present). Consult `references/repositories-and-pids.md` (life-sciences and neuroimaging section, including the standard-driven OpenNeuro exception) and `references/european-services.md`. Recommend EBRAINS as EU primary + OpenNeuro as discipline-standard secondary, with mirror on Zenodo. Justify in DMP terms.

### Example 4 — institutional DMP, no specific funder

> **User**: *"Our university library wants to draft an institutional Research Data Management policy."*

**Expected behaviour**: present `assets/dmp-institutional.md`. Consult `references/fair-principles.md` for the self-assessment section and `references/european-services.md` for the recommended sovereign stack.

### Example 5 — out-of-scope redirect

> **User**: *"DMP for the open data of the municipality of Bologna."*

**Expected behaviour**: do NOT use this skill. Redirect to `governance-dati-pa` (Italian PA regulatory framework, CAD/GDPR/Legislative Decrees).

---

## Available references

Read references only when needed. Do not preload them.

- `references/fair-principles.md` — the 15 FAIR sub-principles with operational criteria and self-assessment checklist
- `references/european-services.md` — **central map** of EU-first services by category; pillar of the sovereignty principle
- `references/funder-requirements.md` — DMP structure required by Horizon Europe, ERC, MSCA, PRIN/MUR, DFG, ANR, NWO, SNSF
- `references/repositories-and-pids.md` — data repositories by domain, DOI providers, certifications
- `references/licensing.md` — Creative Commons, software licences (MIT, Apache, EUPL, GPL), dataset/code/article compatibility
- `references/ci-and-automation.md` — metadata validation pipelines, automated deposit, integration with Codeberg/GitLab CI
- `references/gdpr-research.md` — GDPR for research: Art. 89 derogations, legal basis, consent, pseudonymisation vs anonymisation, DPIA, cross-border transfers, joint controllership, biobank / EHDS specifics
- `references/data-papers.md` — peer-reviewed dataset publications: when to publish, sovereignty-annotated journal map (ESSD, Open Research Europe, RIO, JOHD; Scientific Data, Data in Brief, GigaScience), structure, DOI cross-linking, licence coordination

## Available assets

- `assets/dmp-horizon-europe.md` — DMP template compliant with the Horizon Europe DMP structure (informed by the Science Europe Practical Guide)
- `assets/dmp-institutional.md` — template for generic institutional DMPs (universities, centres, infrastructures)
- `assets/dmp-fair.md` — lightweight template for FAIR contexts without a funder
- `assets/dmp-checklist.md` — operational checklist with tickable items
- `assets/datapackage-template.md` — Frictionless Data Package template (datapackage.json + Table Schema)

## Available scripts

- `scripts/deposit_zenodo.py` — automated deposit on Zenodo via API, idempotent, with `newversion` support

---

## Maintenance notes

The skill is designed to be versioned on git. Update:

- **Funder requirements** when calls change templates (annual check at least).
- **European services map** when new EU-incorporated offerings emerge, or when an EU service changes ownership to a non-European group (e.g. acquisitions, M&A).
- **FAIR principles** when GO FAIR releases new metrics or when EOSC publishes new interoperability requirements.
- **Repositories** when a repository gains/loses CoreTrustSeal certification or when API endpoints change.

---
> Source: [orazionelson/dmp-builder](https://github.com/orazionelson/dmp-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
