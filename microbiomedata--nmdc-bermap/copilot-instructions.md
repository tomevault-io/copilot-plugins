## nmdc-bermap

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a LinkML schema project for representing DOE BER (Biological and Environmental Research) funded research programs: Bioenergy Research Centers (BRCs), Scientific Focus Areas (SFAs), and User Facilities. The schema captures metadata, datasets, publications, and organizational structure.

The knowledge base tracks:
- **BRCs**: GLBRC, JBEI, CABBI, CBI - with studies, datasets, KBase narratives
- **Genomic Science SFAs**: ENIGMA, m-CAFEs, Microbes Persist, PMI, SEED, LLNL Secure Biosystems, etc.
- **ESS SFAs**: Watershed Function, River Corridor, Mercury SFA, WaDE, Wetland SFA, Floodplain SFA, etc.
- **User Facilities**: JGI, EMSL, KBase, ESS-DIVE, NMDC

## Common Commands

All commands use `just` (command runner). Run `just` to see available commands.

```bash
# Install dependencies
just install

# Regenerate Python datamodels from schema
just gen-python

# Run all tests
just test

# Validate the database against schema
just validate-db

# Validate enum ontology mappings
just validate-enums

# Validate publication references (fetches/caches metadata)
just validate-refs

# Generate documentation
just gen-doc

# Convert database to OWL (ABox + TBox)
just gen-abox-tbox

# Fetch BRC datasets from API
just fetch-brc-datasets

# Fetch datasets for a specific center
just fetch-brc-datasets-center GLBRC
```

## Architecture

### Schema-Driven Design
- **Schema source**: `src/nmdc_sfas_brcs/schema/nmdc_sfas_brcs.yaml` - the authoritative LinkML schema
- **Generated datamodels**: `src/nmdc_sfas_brcs/datamodel/` - Python classes auto-generated from schema
- **Database**: `db/sfas-brcs.yaml` - YAML instance data conforming to the schema
- **BRC Datasets**: `db/brc_datasets/{glbrc,jbei,cabbi,cbi}/` - individual YAML files per dataset from BRC API

### Key Schema Classes
- `ResearchProgramCollection` - root container (tree_root)
- `ResearchProgram` - abstract base for BRCs and SFAs
- `BioenergyResearchCenter`, `ScientificFocusArea`, `UserFacility` - program types
- `Study` - research studies with NMDC IDs, BioProject IDs, and ingest priorities
- `Reference` - publications with id (DOI), title, and findings
- `Finding` - key claims from publications with optional supporting_text and reference
- `Dataset` - research datasets with external identifiers (NMDC, JGI, Ameriflux, LTER)
- `KBaseNarrative` - KBase analysis narratives with narrative IDs and URLs

### Reference Validation
The schema uses `linkml-reference-validator` annotations:
- `implements: linkml:authoritative_reference` on `Reference.id` and `Finding.reference`
- `implements: linkml:excerpt` on `Finding.supporting_text`

This enables validation that supporting_text quotes actually appear in cited publications.

### NMDC Integration
Studies can have:
- `nmdc_study_id` - existing NMDC study identifier (e.g., `nmdc:sty-11-r2h77870`)
- `nmdc_ingest_priority` - HIGH, MEDIUM, or LOW priority for NMDC ingest
- `bioproject_ids` - BioProject accessions for sequencing data

### Custom Validators
- `src/nmdc_sfas_brcs/validators/enum_evaluator.py` - validates enum permissible values against ontology mappings

### BRC API Integration
- `src/nmdc_sfas_brcs/scripts/fetch_brc_datasets.py` - fetches datasets from api.bioenergy.org
- Datasets are organized by center in `db/brc_datasets/{center}/`
- Each dataset is stored as an individual YAML file named by its identifier

## Data Editing

Edit `db/sfas-brcs.yaml` directly. After schema changes, regenerate datamodels with `just gen-python` and validate with `just validate-db`.

When adding new studies:
1. Check NMDC for existing study IDs using `nmdc study` CLI
2. Add `nmdc_study_id` if study exists in NMDC
3. Set `nmdc_ingest_priority` for studies with microbiome data not yet in NMDC
4. Include `bioproject_ids` for sequencing studies

## Key Files

- `config.public.mk` - environment variables (LINKML_SCHEMA_NAME, paths)
- `project.justfile` - project-specific just recipes
- `justfile` - base LinkML project recipes (from copier template)
- `db/sfas-brcs.yaml` - main knowledge base
- `db/brc_datasets/` - BRC API dataset cache

---
> Source: [microbiomedata/nmdc-bermap](https://github.com/microbiomedata/nmdc-bermap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
