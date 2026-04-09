## mycosci-github-io

> > **Single source of truth for AI assistants, developers, and contributors.**

# MycoSci Project Documentation

> **Single source of truth for AI assistants, developers, and contributors.**
> Last updated: 2026-01-04

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Business Model](#2-business-model)
3. [Architecture](#3-architecture)
4. [Tech Stack](#4-tech-stack)
5. [Development Setup](#5-development-setup)
6. [Project Structure](#6-project-structure)
7. [Content System](#7-content-system)
8. [Templates](#8-templates)
9. [Data Entry Guide](#9-data-entry-guide)
10. [WordPress Backend](#10-wordpress-backend)
11. [API Documentation](#11-api-documentation)
12. [Database Schemas](#12-database-schemas)
13. [AI Detection System](#13-ai-detection-system)
14. [Mobile Apps](#14-mobile-apps)
15. [Deployment](#15-deployment)
16. [Contributing](#16-contributing)

---

## 1. Project Overview

**MycoSci** is an open-source mycology platform combining:
- Free educational content (900+ documented mushroom species)
- E-commerce (lab supplies, cultures, agar products)
- Gene management service (cryo storage, sample tracking)
- AI-powered mushroom detection (offline-capable)
- Interactive data exploration (MycoGram, phylogenetic trees)

**Live Site:** https://mycosci.com
**Repository:** https://github.com/MycoSci/mycosci.github.io

---

## 2. Business Model

### Revenue Streams

| Product/Service | Description | Pricing Model |
|-----------------|-------------|---------------|
| **Agar Products** | Premade mixes, ready-poured plates, colonized plates | Per-unit sales |
| **Liquid Culture** | Syringes, jars, starter kits | Per-unit sales |
| **Lab Supplies** | Petri dishes, scalpels, sterilization equipment | Per-unit sales |
| **Gene Storage** | Cryo preservation of customer samples | $15/month subscription |
| **Source Pulls** | Request culture from stored samples | $25/pull + shipping |
| **Genetic Sequencing** | ITS/LSU sequencing of samples | $75/sample |

### Gene Management Service

Partnership with university cleanroom lab for:
- Long-term cryo storage (-196°C liquid nitrogen)
- Annual viability testing
- Genetic marker sequencing
- Consistent source material for growers

**Target Scale:** 100-1000 samples (medium scale)

### Free Offerings

- 900+ species database with full documentation
- AI mushroom detection (web, iOS, Android - offline capable)
- Interactive taxonomy exploration
- MycoGram visual feed
- Phylogenetic tree visualizations

---

## 3. Architecture

### System Diagram

```
                              [USERS]
                                 |
            +--------------------+--------------------+
            |                    |                    |
       Web Browser           iOS App            Android App
            |                    |                    |
            v                    v                    v
+-----------------------------------------------------------------------+
|                         CLOUDFLARE (CDN/Edge)                         |
+-----------------------------------------------------------------------+
                                 |
        +------------------------+------------------------+
        |                        |                        |
        v                        v                        v
+---------------+       +------------------+      +------------------+
|   VERCEL      |       |   WORDPRESS      |      |   CLOUDFLARE R2  |
|   (Astro.js)  |<----->|   (Headless)     |      |   (Storage)      |
+---------------+       +------------------+      +------------------+
|               |       |                  |      |                  |
| - SSG pages   |       | - WooCommerce    |      | - AI models      |
| - SSR routes  |       | - WPGraphQL      |      | - Images         |
| - API routes  |       | - Gene service   |      | - User uploads   |
| - MDX content |       | - User accounts  |      |                  |
+---------------+       +------------------+      +------------------+
                               |
                        +------+------+
                        |   MySQL     |
                        | (Gene DB)   |
                        +-------------+
```

### Domain Structure

| Domain | Purpose | Host |
|--------|---------|------|
| `mycosci.com` | Main frontend | Vercel |
| `api.mycosci.com` | WordPress REST/GraphQL | Kinsta/WPEngine |
| `cdn.mycosci.com` | Static assets, AI models | Cloudflare R2 |

---

## 4. Tech Stack

### Frontend

| Technology | Version | Purpose |
|------------|---------|---------|
| Astro | 5.6.1 | Static site generator with SSR |
| Starlight | 0.32.3 | Documentation theme |
| React | 18.x | Interactive components |
| TailwindCSS | 3.x | Styling |
| TensorFlow.js | 4.x | Client-side AI inference |
| D3.js | 7.x | Data visualizations |

### Backend (WordPress Headless)

| Technology | Purpose |
|------------|---------|
| WordPress | Content management |
| WooCommerce | E-commerce |
| WPGraphQL | API layer |
| ACF Pro | Custom fields |
| MySQL 8.0 | Database |

### Mobile

| Technology | Purpose |
|------------|---------|
| React Native | Cross-platform apps |
| Expo | Build toolchain |
| TensorFlow Lite | On-device AI |

### Infrastructure

| Service | Purpose |
|---------|---------|
| Vercel | Frontend hosting |
| Kinsta/WPEngine | WordPress hosting |
| Cloudflare R2 | Object storage |
| Cloudflare | CDN, DNS, security |
| Stripe | Payments |
| Amazon SES | Transactional email |

---

## 5. Development Setup

### Prerequisites

- Node.js v18+ (v24.4.1 recommended)
- npm v9+
- Git

### Quick Start

```bash
# Clone repository
git clone https://github.com/MycoSci/mycosci.github.io.git
cd mycosci.github.io

# Install dependencies
npm install

# Start development server
npm run dev
# Site available at http://localhost:4321
```

### Available Commands

| Command | Action |
|---------|--------|
| `npm install` | Install dependencies |
| `npm run dev` | Start dev server at `localhost:4321` |
| `npm run build` | Build production site to `./dist/` |
| `npm run preview` | Preview built site locally |

### Environment Variables

Create `.env` for local development:

```env
# WordPress API (when backend is set up)
WP_GRAPHQL_ENDPOINT=https://api.mycosci.com/graphql
WP_REST_ENDPOINT=https://api.mycosci.com/wp-json

# Public URLs
PUBLIC_SITE_URL=https://mycosci.com
PUBLIC_CDN_URL=https://cdn.mycosci.com
PUBLIC_MODEL_VERSION=v1

# Stripe (for e-commerce)
STRIPE_PUBLIC_KEY=pk_test_xxx
```

---

## 6. Project Structure

```
mycosci.github.io/
├── public/                    # Static assets (favicon, images)
├── src/
│   ├── components/            # Reusable components (future)
│   │   ├── ai/               # AI detection components
│   │   ├── commerce/         # Shop components
│   │   ├── gene-service/     # Dashboard components
│   │   ├── mycogram/         # Feed components
│   │   └── taxonomy/         # Tree visualizations
│   ├── content/
│   │   └── docs/             # All MDX documentation
│   │       ├── index.mdx     # Homepage
│   │       ├── Guides/       # Getting started guides
│   │       ├── Taxonomy/     # 905 species files (hierarchical)
│   │       ├── Cultivation/  # Growing techniques
│   │       ├── lab/          # Lab protocols
│   │       ├── Equipment/    # Gear guides
│   │       ├── Foraging/     # Field safety
│   │       ├── chemistry-nutrition/
│   │       ├── recipes/
│   │       ├── references/   # Research library
│   │       ├── blog/         # Articles
│   │       ├── gallery/
│   │       ├── mycogram/
│   │       └── resources/
│   ├── data/                  # JSON data files
│   │   ├── species.json
│   │   ├── authors.json
│   │   └── posts.json
│   ├── lib/                   # Utility modules (future)
│   │   ├── wordpress.ts      # GraphQL client
│   │   ├── ai-model.ts       # TF.js helpers
│   │   └── auth.ts           # JWT auth
│   ├── pages/                 # Route pages (future SSR)
│   │   ├── shop/
│   │   ├── gene-service/
│   │   ├── detect/
│   │   ├── mycogram/
│   │   └── account/
│   └── content.config.ts      # Content collection schema
├── data/
│   └── species.xlsx           # Master species spreadsheet
├── .github/
│   └── workflows/
│       └── astro.yml          # CI/CD pipeline
├── astro.config.mjs           # Astro configuration
├── starlight.config.mjs       # Sidebar & theme config
├── tsconfig.json              # TypeScript config
├── package.json               # Dependencies & scripts
└── claude.md                  # This file
```

---

## 7. Content System

### Content Collections

All content lives in `src/content/docs/` as Markdown (`.md`) or MDX (`.mdx`) files.

**Schema** (`src/content.config.ts`):
```typescript
import { defineCollection } from 'astro:content';
import { docsSchema } from '@astrojs/starlight/schema';

export const collections = {
  docs: defineCollection({
    schema: docsSchema({
      extend: (context) => context.schema.extend({
        author: z.string().optional(),
        pubDate: z.date().optional(),
        tags: z.array(z.string()).optional(),
      }),
    }),
  }),
};
```

### Taxonomy Structure

Species are organized hierarchically:
```
Taxonomy/
├── Ascomycota/
│   └── [Class]/[Order]/[Family]/[Genus]/[Species].mdx
└── Basidiomycota/
    └── [Class]/[Order]/[Family]/[Genus]/[Species].mdx
```

Each taxonomic rank has an `index.mdx` with `sidebar.order: -1` to appear first.

### Starlight Components

Available in MDX files:

```mdx
import { Card, CardGrid, Tabs, TabItem, Badge, Aside, Steps, LinkButton } from '@astrojs/starlight/components';

<Card title="Title" icon="sparkles">Content</Card>
<CardGrid stagger>...</CardGrid>
<Tabs><TabItem label="Tab 1">...</TabItem></Tabs>
<Badge text="edible" variant="success" />
<Aside type="caution" title="Warning">...</Aside>
<Steps>1. Step one 2. Step two</Steps>
<LinkButton href="/path" variant="primary">Click</LinkButton>
```

**Badge Variants:** `success`, `default`, `danger`, `tip`, `note`
**Aside Types:** `note`, `tip`, `caution`, `danger`

---

## 8. Templates

### Species Page Template

Use this template for new species entries. Replace all `{{placeholder}}` values.

```mdx
---
title: "{{common_name}} ({{genus}} {{species}})"
description: "{{description}}"
---

# *{{genus}} {{species}}*

<CardGrid stagger>
  <Card title="Quick Facts" icon="sparkles">
    - <Badge text="{{edibility_status}}" variant="{{variant}}" />
    - **Cap**: {{cap_summary}}
    - **Gills / Ridges / Pores**: {{gill_summary}}
    - **Spore Print**: {{spore_print_color}}
  </Card>
  <Card title="Habitat Snapshot" icon="map">
    - **Substrate**: {{substrate}}
    - **Habitat**: {{habitat}}
    - **Range**: {{geographic_distribution}}
    - **Season**: {{fruiting_season}}
  </Card>
</CardGrid>

<Aside type="caution" title="Safety Note">
  {{safety_note}}
</Aside>

## 1. Scientific Taxonomy

| Rank | Name |
|------|------|
| Kingdom | Fungi |
| Division / Phylum | {{division}} |
| Class | {{class}} |
| Order | {{order}} |
| Family | {{family}} |
| Genus | *{{genus}}* |
| Species | *{{species}}* |
| Authority | {{taxonomic_authority}} |
| Synonyms | {{synonyms}} |

## 2. Discovery & Naming

- **Discoverer / Describer:** {{discoverer}}
- **Year Described:** {{description_year}}
- **Type Locality:** {{type_locality}}
- **Type Specimen ID:** {{type_specimen_id}}
- **Etymology:** {{etymology}}
- **Historical Notes:** {{historical_notes}}

## 3. Morphological Characteristics

<Tabs syncKey="morphology">
  <TabItem label="Macroscopic" icon="eye">

  ### Cap
  - **Shape:** {{cap_shape}}
  - **Size:** {{cap_size}} cm
  - **Color:** {{cap_color}}
  - **Surface:** {{cap_surface}}

  ### Gills / Ridges / Pores
  - **Type:** {{gill_type}}
  - **Attachment:** {{gill_attachment}}
  - **Color:** {{gill_color}}

  ### Stem
  - **Present:** {{stem_present}}
  - **Dimensions:** {{stem_dimensions}} cm
  - **Characteristics:** {{stem_characteristics}}

  ### Flesh
  - **Color & Changes:** {{flesh_characteristics}}
  - **Odor:** {{odor}}
  - **Taste:** {{taste}}

  ### Spore Print
  - **Color:** {{spore_print_color}}

  </TabItem>

  <TabItem label="Microscopic" icon="microscope">
  - **Spore Dimensions:** {{spore_dimensions}} um
  - **Spore Shape:** {{spore_shape}}
  - **Ornamentation:** {{spore_ornamentation}}
  - **Basidia / Cystidia:** {{basidia_cystidia}}
  </TabItem>
</Tabs>

## 4. Chemical Profile

- **Toxic Compounds:** {{toxic_compounds}}
- **Medicinal Compounds:** {{medicinal_compounds}}
- **Nutritional Snapshot:** {{nutritional_summary}}
- **Secondary Metabolites:** {{secondary_metabolites}}
- **Chemical Tests:** {{chemical_reactions}}

## 5. DNA & Genetics

| Marker | Accession |
|--------|-----------|
| ITS | {{its_accession}} |
| LSU | {{lsu_accession}} |
| SSU | {{ssu_accession}} |
| Additional | {{other_markers}} |

- **Genome Assembly:** {{genome_reference}}
- **Voucher Specimen:** {{voucher_specimen}}

## 6. Ecology & Habitat

- **Habitat:** {{habitat}}
- **Substrate:** {{substrate}}
- **Symbiosis / Trophic Type:** {{trophic_type}}
- **Ecological Role:** {{ecological_role}}
- **Distribution:** {{geographic_distribution}}
- **Altitude Range:** {{altitude_range}}
- **Population Status:** {{population_status}}

## 7. Environmental Tolerances

- **Temperature Range:** {{temperature_range}} C
- **Moisture Needs:** {{moisture_requirements}}
- **pH Range:** {{ph_range}}
- **Light:** {{light_requirements}}
- **USDA Zones:** {{usda_zones}}

## 8. Culinary Information

<Aside type="tip" title="Culinary Overview">
  {{culinary_overview}}
</Aside>

- **Common Uses:** {{culinary_uses}}
- **Preparation Requirements:** {{preparation_requirements}}
- **Flavor & Texture:** {{flavor_texture}}
- **Nutritional Value:** {{nutritional_value}}
- **Cautions:** {{culinary_cautions}}

## 9. Toxicity & Safety

<Card title="Toxicity Details" icon="shield">
  **Status:** {{toxicity_status}}
  **Toxins:** {{toxin_compounds}}
  **Symptoms:** {{poisoning_symptoms}}
  **Treatment:** {{treatment}}
</Card>

## 10. Cultural & Historical Significance

- **Common Names:** {{common_names}}
- **Folklore & Mythology:** {{folklore}}
- **Traditional Uses:** {{traditional_uses}}
- **Historic References:** {{historic_references}}

## 11. Dispersal & Restoration

- **Spore Dispersal:** {{spore_dispersal}}
- **Reproduction Mode:** {{reproduction_mode}}
- **Cultivation Difficulty:** {{cultivation_difficulty}}
- **Restoration Protocols:** {{restoration_protocols}}
- **Conservation Notes:** {{conservation_notes}}

## 12. Similar Species & Misidentifications

| Look-alike | Key Differences | Risk |
|------------|----------------|------|
| {{similar_species_1}} | {{differences_1}} | {{risk_1}} |
| {{similar_species_2}} | {{differences_2}} | {{risk_2}} |

## 13. Conservation Status

- **IUCN:** {{iucn_status}}
- **Regional Status:** {{regional_status}}
- **Threats:** {{threats}}
- **Population Trend:** {{population_trend}}
- **Actions:** {{conservation_actions}}

## 14. Field Identification Rubric

<Steps>
1. {{id_step1}}
2. {{id_step2}}
3. {{id_step3}}
4. {{id_step4}}
</Steps>

<Aside type="note" title="Diagnostic Summary">
  {{diagnostic_summary}}
</Aside>

## 15. References

- {{reference1}}
- {{reference2}}
- {{reference3}}

<LinkButton href="#top" variant="minimal" icon="arrow-up" iconPlacement="start">Back to top</LinkButton>
```

### Index Page Template

For category/section landing pages:

```mdx
---
title: "{{section_title}}"
description: "{{section_description}}"
sidebar:
  order: -1
---

# {{section_title}}

{{intro_paragraph}}

## Contents

<CardGrid>
  <Card title="{{subsection_1}}" icon="document">
    {{subsection_1_description}}
  </Card>
  <Card title="{{subsection_2}}" icon="document">
    {{subsection_2_description}}
  </Card>
</CardGrid>
```

---

## 9. Data Entry Guide

### Species Field Reference

When creating species entries, use these field definitions:

#### Basic Identifiers

| Field | Description | Example |
|-------|-------------|---------|
| `genus` | Latin genus name (capitalized) | `Cantharellus` |
| `species` | Specific epithet (lowercase) | `cibarius` |
| `description` | One-sentence summary for frontmatter | `Comprehensive profile for...` |
| `common_names` | Comma-separated vernacular names | `Golden Chanterelle, Girolle (FR)` |

#### Quick Facts Card

| Field | Description | Example |
|-------|-------------|---------|
| `edibility_status` | `edible`, `inedible`, `toxic`, `psychoactive`, `unknown` | `edible` |
| `variant` | Badge color (computed): `success`, `default`, `danger`, `tip`, `note` | `success` |
| `cap_summary` | Short phrase: shape, size, color | `Convex to funnel, 3-10 cm, golden-yellow` |
| `gill_summary` | Edge surface description | `Decurrent ridges, thick, same color as cap` |
| `spore_print_color` | Color of massed spores | `Creamy-white` |

#### Habitat Snapshot

| Field | Description | Example |
|-------|-------------|---------|
| `substrate` | What it grows on | `Acidic soils in mossy forests` |
| `habitat` | General ecosystem | `Coniferous & mixed woodlands` |
| `geographic_distribution` | Continents/countries | `Europe, N. America, parts of Asia` |
| `fruiting_season` | Months or season | `Jun - Oct (after rains)` |

#### Taxonomy Section

| Field | Description | Example |
|-------|-------------|---------|
| `division` | Fungal phylum | `Basidiomycota` |
| `class` | Taxonomic class | `Agaricomycetes` |
| `order` | Order name | `Cantharellales` |
| `family` | Family name | `Cantharellaceae` |
| `taxonomic_authority` | Author citation | `Fr. (1821)` |
| `synonyms` | Other scientific names | `Agaricus cibarius (Fr., 1821)` |

#### Discovery & Naming

| Field | Description | Example |
|-------|-------------|---------|
| `discoverer` | Person who first described | `Elias Magnus Fries` |
| `description_year` | Year of original description | `1821` |
| `type_locality` | Where holotype was collected | `Oland, Sweden` |
| `type_specimen_id` | Herbarium accession | `UPS:Fungi:9967` |
| `etymology` | Origin of name | `"cibarius" - Latin for "food"` |

#### Morphology - Macroscopic

| Field | Description | Example |
|-------|-------------|---------|
| `cap_shape` | Shape at maturity | `Convex to funnel` |
| `cap_size` | Diameter range (cm) | `3-10` |
| `cap_color` | Typical colors | `Deep golden-yellow to egg-yolk orange` |
| `cap_surface` | Texture/finish | `Smooth, matte; dry when wet` |
| `gill_type` | `True gills`, `ridges`, or `pores` | `Ridges (false gills)` |
| `gill_attachment` | How they join stem | `Decurrent - running down stem` |
| `gill_color` | Fresh color | `Paler yellow to same as cap` |
| `stem_present` | `Yes` or `No` | `Yes` |
| `stem_dimensions` | Length x thickness (cm) | `3-8 x 1-2` |
| `stem_characteristics` | Surface, ring, volva | `Solid, smooth, same color` |
| `flesh_characteristics` | Color & bruising | `Whitish to light yellow; does not bruise` |
| `odor` | Smell description | `Distinct apricot or fruity` |
| `taste` | Flavor (if safe) | `Mild, peppery raw; pleasant cooked` |

#### Morphology - Microscopic

| Field | Description | Example |
|-------|-------------|---------|
| `spore_dimensions` | Length x width (um) | `7-10 x 5-6` |
| `spore_shape` | Shape description | `Ellipsoid, smooth, thin-walled` |
| `spore_ornamentation` | Surface details | `Smooth, non-amyloid` |
| `basidia_cystidia` | Basidia and cystidia | `Basidia 4-spored; cystidia absent` |

#### DNA & Genetics

| Field | Description | Example |
|-------|-------------|---------|
| `its_accession` | GenBank ITS accession | `KF291741` |
| `lsu_accession` | LSU marker accession | `JX869105` |
| `ssu_accession` | SSU marker accession | `EU784293` |
| `other_markers` | Additional markers | `TEF1-a: MW461198` |
| `genome_reference` | Link/citation to assembly | `Draft assemblies in progress` |
| `voucher_specimen` | Physical voucher ID | `UPS:Fungi:9967 (holotype)` |

#### Ecology

| Field | Description | Example |
|-------|-------------|---------|
| `trophic_type` | Nutritional mode | `Ectomycorrhizal (mutualistic)` |
| `ecological_role` | Ecosystem function | `Enhances nutrient exchange...` |
| `altitude_range` | Elevation band | `Sea level - 2000 m` |
| `population_status` | Commonality | `Abundant where habitat intact` |

#### Environmental Tolerances

| Field | Description | Example |
|-------|-------------|---------|
| `temperature_range` | Growth/fruiting window (C) | `10-25` |
| `moisture_requirements` | Humidity needs | `Requires high soil moisture` |
| `ph_range` | Soil/substrate pH | `4-6` |
| `light_requirements` | Light preference | `Shaded understory` |
| `usda_zones` | Hardiness zones | `3-8` |

#### Toxicity & Safety

| Field | Description | Example |
|-------|-------------|---------|
| `toxicity_status` | Category | `Non-toxic`, `Mildly toxic`, `Deadly` |
| `toxin_compounds` | Names of toxins | `None identified` |
| `poisoning_symptoms` | Clinical signs | `GI upset, liver failure...` |
| `treatment` | First aid/medical | `Supportive care; activated charcoal` |

#### Look-alikes

| Field | Description | Example |
|-------|-------------|---------|
| `similar_species_1` | Scientific + common name | `Omphalotus olearius (Jack O'Lantern)` |
| `differences_1` | Key differences | `True gills vs. ridges` |
| `risk_1` | Severity | `Toxic` |

#### Conservation

| Field | Description | Example |
|-------|-------------|---------|
| `iucn_status` | Red List code | `LC`, `EN`, `CR` |
| `regional_status` | National protections | `Protected in Scandinavia` |
| `threats` | Main threats | `Habitat loss, clear-cutting` |
| `population_trend` | Trend direction | `Stable` |
| `conservation_actions` | Measures | `Habitat preservation...` |

### Data Entry Tips

- Keep each cell plain text, no markdown inside CSV/Excel
- Use commas to separate multiple values
- Use en-dash (-) for ranges
- Include units (cm, um) where appropriate
- Match capitalization and spelling consistently
- Compute `variant` from `edibility_status`:
  - `edible` -> `success`
  - `inedible` -> `default`
  - `toxic` -> `danger`
  - `psychoactive` -> `tip`
  - `unknown` -> `note`

---

## 10. WordPress Backend

### Required Plugins

| Plugin | Purpose |
|--------|---------|
| WooCommerce | E-commerce core |
| WPGraphQL | GraphQL API |
| WPGraphQL for WooCommerce | E-commerce API |
| WPGraphQL JWT Authentication | Secure auth |
| Advanced Custom Fields PRO | Custom fields |
| WPGraphQL for ACF | Expose ACF via GraphQL |
| WooCommerce Subscriptions | Gene storage billing |
| WP Mail SMTP | Email via Amazon SES |

### Custom Post Types

```php
// Gene Samples
register_post_type('gene_sample', [
    'label' => 'Gene Samples',
    'public' => false,
    'show_ui' => true,
    'show_in_graphql' => true,
    'graphql_single_name' => 'geneSample',
    'graphql_plural_name' => 'geneSamples',
]);

// Cryo Locations
register_post_type('cryo_location', [
    'label' => 'Cryo Locations',
    'public' => false,
    'show_ui' => true,
    'show_in_graphql' => true,
]);

// Lab Partners
register_post_type('lab_partner', [
    'label' => 'Lab Partners',
    'public' => true,
    'show_in_graphql' => true,
]);

// MycoGram Posts
register_post_type('mycogram_post', [
    'label' => 'MycoGram Posts',
    'public' => true,
    'show_in_graphql' => true,
    'supports' => ['title', 'editor', 'thumbnail', 'author', 'comments'],
]);
```

### Custom Taxonomies

```php
// Sample Status: received, processing, stored, pulled, expired, contaminated
register_taxonomy('sample_status', 'gene_sample', [...]);

// Storage Type: cryo_vial, agar_slant, liquid_culture, spore_syringe
register_taxonomy('storage_type', 'gene_sample', [...]);

// Fungal Species (links samples to species)
register_taxonomy('fungal_species', ['gene_sample', 'mycogram_post'], [...]);
```

### ACF Field Groups

**Gene Sample Fields:**
- `sample_id` (auto-generated: MYCO-2025-00001)
- `customer_id` (relationship to user)
- `species_name`, `species_taxonomy`
- `collection_date`, `received_date`
- `source_location`, `source_coordinates`
- `isolation_method` (tissue, spore, liquid_culture, agar)
- `cryo_location_id`, `storage_position`
- `viability_status`, `last_viability_check`
- `genetic_markers` (repeater: marker_type, accession, sequence_file)
- `pull_requests` (repeater: date, quantity, destination)

**Cryo Location Fields:**
- `dewar_id`, `nickname`
- `rack_count`, `box_count_per_rack`, `position_count_per_box`
- `temperature` (-196 for LN2)
- `lab_partner`, `address`
- `capacity_total`, `capacity_used`

---

## 11. API Documentation

### GraphQL Endpoint

**URL:** `https://api.mycosci.com/graphql`

### Authentication

```graphql
mutation Login($username: String!, $password: String!) {
  login(input: { username: $username, password: $password }) {
    authToken
    refreshToken
    user {
      id
      name
      email
    }
  }
}
```

Include token in subsequent requests:
```
Authorization: Bearer <authToken>
```

### Key Queries

```graphql
# Get MycoGram Feed
query MycogramFeed($first: Int!, $after: String) {
  mycogramPosts(first: $first, after: $after) {
    nodes {
      id
      title
      featuredImage { sourceUrl }
      species { slug, title }
      author { name, avatar { url } }
      likesCount
    }
    pageInfo { hasNextPage, endCursor }
  }
}

# Get Customer's Gene Samples
query MyGeneSamples {
  geneSamples(where: { author: $userId }) {
    nodes {
      sampleId
      speciesName
      storagePosition
      viabilityStatus
      lastViabilityCheck
    }
  }
}

# Get Products by Category
query ShopProducts($category: String!) {
  products(where: { category: $category }) {
    nodes {
      id
      name
      price
      image { sourceUrl }
    }
  }
}
```

### Key Mutations

```graphql
# Submit Gene Sample
mutation SubmitGeneSample($input: GeneSampleInput!) {
  createGeneSample(input: $input) {
    geneSample {
      sampleId
      status
    }
  }
}

# Request Sample Pull
mutation RequestPull($sampleId: String!, $quantity: Int!) {
  createPullRequest(input: {
    sampleId: $sampleId
    quantity: $quantity
  }) {
    pullRequest {
      id
      status
      trackingNumber
    }
  }
}

# Create MycoGram Post
mutation CreateMycogramPost($input: MycogramPostInput!) {
  createMycogramPost(input: $input) {
    mycogramPost {
      id
      title
    }
  }
}
```

### Astro Integration

```typescript
// src/lib/wordpress.ts
import { GraphQLClient } from 'graphql-request';

const client = new GraphQLClient(
  import.meta.env.WP_GRAPHQL_ENDPOINT,
  { headers: { 'Content-Type': 'application/json' } }
);

export async function getMycogramFeed(limit = 20) {
  const query = `
    query MycogramFeed($first: Int!) {
      mycogramPosts(first: $first) {
        nodes { id, title, featuredImage { sourceUrl } }
      }
    }
  `;
  return client.request(query, { first: limit });
}
```

---

## 12. Database Schemas

### Gene Management Tables

```sql
-- Gene Samples (Main table)
CREATE TABLE wp_mycosci_gene_samples (
    sample_id VARCHAR(20) PRIMARY KEY,      -- MYCO-2025-00001
    wp_post_id BIGINT UNSIGNED,
    customer_id BIGINT UNSIGNED NOT NULL,
    subscription_id BIGINT UNSIGNED,

    -- Species
    species_name VARCHAR(255) NOT NULL,
    species_taxonomy_id BIGINT UNSIGNED,
    common_name VARCHAR(255),

    -- Collection
    collection_date DATE,
    received_date DATE,
    source_location TEXT,
    source_lat DECIMAL(10, 8),
    source_lng DECIMAL(11, 8),
    isolation_method ENUM('tissue', 'spore', 'liquid_culture', 'agar'),

    -- Storage
    cryo_location_id BIGINT UNSIGNED,
    storage_position VARCHAR(50),           -- D1-R3-B2-P15
    storage_date DATE,

    -- Status
    status ENUM('pending', 'received', 'processing', 'stored', 'pulled', 'expired', 'contaminated'),
    viability_status ENUM('viable', 'questionable', 'non_viable', 'untested'),
    last_viability_check DATE,

    -- Metadata
    notes TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_customer (customer_id),
    INDEX idx_status (status)
);

-- Cryo Storage Locations
CREATE TABLE wp_mycosci_cryo_locations (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    dewar_id VARCHAR(20) NOT NULL,
    nickname VARCHAR(100),

    rack_count INT DEFAULT 10,
    box_count_per_rack INT DEFAULT 10,
    position_count_per_box INT DEFAULT 81,  -- 9x9 grid

    temperature DECIMAL(5,2) DEFAULT -196.00,
    lab_partner_id BIGINT UNSIGNED,
    address TEXT,

    capacity_total INT,
    capacity_used INT DEFAULT 0,
    last_fill_date DATE,

    UNIQUE KEY idx_dewar (dewar_id)
);

-- Lab Partners
CREATE TABLE wp_mycosci_lab_partners (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    institution_name VARCHAR(255) NOT NULL,
    institution_type ENUM('university', 'commercial', 'government', 'nonprofit'),

    primary_contact_name VARCHAR(255),
    primary_contact_email VARCHAR(255),
    primary_contact_phone VARCHAR(50),

    address_street VARCHAR(255),
    address_city VARCHAR(100),
    address_state VARCHAR(50),
    address_zip VARCHAR(20),
    address_country VARCHAR(100),

    capabilities SET('cryo_storage', 'sequencing', 'culturing', 'microscopy'),

    agreement_start DATE,
    agreement_end DATE,
    billing_rate DECIMAL(10,2),

    is_active BOOLEAN DEFAULT TRUE
);

-- Sample Pull Requests
CREATE TABLE wp_mycosci_pull_requests (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    sample_id VARCHAR(20) NOT NULL,
    customer_id BIGINT UNSIGNED NOT NULL,

    request_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    requested_quantity INT DEFAULT 1,
    destination_address TEXT,
    shipping_method ENUM('standard', 'expedited', 'overnight'),

    status ENUM('pending', 'approved', 'processing', 'shipped', 'delivered', 'cancelled'),
    processed_by BIGINT UNSIGNED,
    processed_date TIMESTAMP,

    tracking_number VARCHAR(100),
    carrier VARCHAR(50),
    shipped_date TIMESTAMP,

    order_id BIGINT UNSIGNED,  -- WooCommerce order
    amount DECIMAL(10,2),

    INDEX idx_sample (sample_id),
    INDEX idx_status (status)
);

-- Viability Tests
CREATE TABLE wp_mycosci_viability_tests (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    sample_id VARCHAR(20) NOT NULL,
    test_date DATE NOT NULL,

    test_type ENUM('growth_test', 'visual_inspection', 'staining', 'contamination_check'),
    result ENUM('viable', 'questionable', 'non_viable'),

    growth_rate VARCHAR(50),
    contamination_detected BOOLEAN DEFAULT FALSE,
    contamination_type VARCHAR(100),

    tested_by BIGINT UNSIGNED,
    notes TEXT,
    images JSON,

    INDEX idx_sample_date (sample_id, test_date)
);

-- Genetic Markers
CREATE TABLE wp_mycosci_genetic_markers (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    sample_id VARCHAR(20) NOT NULL,

    marker_type ENUM('ITS', 'LSU', 'SSU', 'RPB1', 'RPB2', 'TEF1', 'OTHER'),
    accession_number VARCHAR(50),
    sequence_file_url VARCHAR(255),
    sequence_length INT,

    analyzed_by VARCHAR(255),
    analysis_date DATE,

    INDEX idx_marker (marker_type, accession_number)
);
```

### Sample ID Generation

```sql
-- Sequence table for auto-increment
CREATE TABLE wp_mycosci_sample_sequence (
    year YEAR PRIMARY KEY,
    last_number INT DEFAULT 0
);

-- Stored procedure
DELIMITER //
CREATE PROCEDURE generate_sample_id(OUT new_id VARCHAR(20))
BEGIN
    DECLARE current_year YEAR DEFAULT YEAR(CURDATE());
    DECLARE next_num INT;

    INSERT INTO wp_mycosci_sample_sequence (year, last_number)
    VALUES (current_year, 1)
    ON DUPLICATE KEY UPDATE last_number = last_number + 1;

    SELECT last_number INTO next_num
    FROM wp_mycosci_sample_sequence
    WHERE year = current_year;

    SET new_id = CONCAT('MYCO-', current_year, '-', LPAD(next_num, 5, '0'));
END //
DELIMITER ;
```

---

## 13. AI Detection System

### Training Data Pipeline

**Data Sources:**
1. iNaturalist API (research-grade observations)
2. GBIF occurrence data
3. MycoGram user uploads (future)

**Target:** 100+ images per species for top 500 species

### Model Architecture

```python
import tensorflow as tf
from tensorflow.keras.applications import MobileNetV3Large

# Base: MobileNetV3 (optimized for mobile)
base_model = MobileNetV3Large(
    input_shape=(224, 224, 3),
    include_top=False,
    weights='imagenet'
)

model = tf.keras.Sequential([
    base_model,
    tf.keras.layers.GlobalAveragePooling2D(),
    tf.keras.layers.Dropout(0.3),
    tf.keras.layers.Dense(512, activation='relu'),
    tf.keras.layers.Dropout(0.3),
    tf.keras.layers.Dense(NUM_SPECIES, activation='softmax')
])
```

### Export Formats

| Format | Size | Platform |
|--------|------|----------|
| TensorFlow.js | ~20MB | Web |
| TFLite (quantized) | ~8MB | Android |
| Core ML | ~15MB | iOS |

### CDN Model Hosting

```
cdn.mycosci.com/models/v1/mushroom-detector/
├── model.json           # TF.js manifest
├── group1-shard1of4.bin # Weights
├── group1-shard2of4.bin
├── group1-shard3of4.bin
├── group1-shard4of4.bin
├── labels.json          # Species mapping
└── metadata.json        # Version info
```

### Client-Side Integration

```typescript
// src/lib/ai-model.ts
import * as tf from '@tensorflow/tfjs';

let model: tf.GraphModel | null = null;
let speciesLabels: string[] = [];

export async function loadModel() {
  if (model) return model;

  model = await tf.loadGraphModel(
    `${import.meta.env.PUBLIC_CDN_URL}/models/v1/mushroom-detector/model.json`
  );

  const response = await fetch(
    `${import.meta.env.PUBLIC_CDN_URL}/models/v1/mushroom-detector/labels.json`
  );
  speciesLabels = await response.json();

  return model;
}

export async function detectMushroom(imageElement: HTMLImageElement) {
  const model = await loadModel();

  const tensor = tf.browser.fromPixels(imageElement)
    .resizeNearestNeighbor([224, 224])
    .toFloat()
    .div(255.0)
    .expandDims();

  const predictions = await model.predict(tensor) as tf.Tensor;
  const probabilities = await predictions.data();

  const results = Array.from(probabilities)
    .map((prob, idx) => ({
      species: speciesLabels[idx],
      confidence: prob,
    }))
    .sort((a, b) => b.confidence - a.confidence)
    .slice(0, 5);

  tensor.dispose();
  predictions.dispose();

  return results;
}
```

---

## 14. Mobile Apps

### Technology Stack

- **Framework:** React Native + Expo
- **Navigation:** Expo Router
- **AI:** TensorFlow Lite (bundled model)
- **Storage:** AsyncStorage / MMKV
- **API:** Shared GraphQL client

### App Structure

```
mobile/
├── app/                      # Expo Router pages
│   ├── (tabs)/
│   │   ├── index.tsx        # Home/Feed
│   │   ├── detect.tsx       # AI Detection
│   │   ├── shop.tsx         # E-commerce
│   │   ├── mycogram.tsx     # Social feed
│   │   └── account.tsx      # User account
│   ├── detect/
│   │   ├── camera.tsx       # Live detection
│   │   └── upload.tsx       # Photo upload
│   ├── gene-service/
│   │   ├── index.tsx        # Dashboard
│   │   └── submit.tsx       # Sample submission
│   └── species/[slug].tsx   # Species detail
├── components/
├── hooks/
├── lib/
│   ├── api.ts               # GraphQL client
│   └── tensorflow.ts        # TFLite bridge
└── assets/
    └── models/
        └── mushroom_detector.tflite
```

### Offline Detection

```typescript
// hooks/useOfflineDetection.ts
import NetInfo from '@react-native-community/netinfo';
import AsyncStorage from '@react-native-async-storage/async-storage';

export function useOfflineDetection() {
  const [isOffline, setIsOffline] = useState(false);

  useEffect(() => {
    const unsubscribe = NetInfo.addEventListener(state => {
      setIsOffline(!state.isConnected);
      if (state.isConnected) syncPendingDetections();
    });
    return unsubscribe;
  }, []);

  const saveDetectionLocally = async (detection) => {
    const pending = await AsyncStorage.getItem('pending_detections');
    const detections = pending ? JSON.parse(pending) : [];
    detections.push({ ...detection, timestamp: Date.now() });
    await AsyncStorage.setItem('pending_detections', JSON.stringify(detections));
  };

  return { isOffline, saveDetectionLocally };
}
```

---

## 15. Deployment

### Astro (Vercel)

**Automatic deployment on push to `main` branch.**

```json
// vercel.json
{
  "framework": "astro",
  "buildCommand": "npm run build",
  "outputDirectory": "dist",
  "headers": [
    {
      "source": "/models/(.*)",
      "headers": [
        { "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }
      ]
    }
  ]
}
```

### Current GitHub Actions Pipeline

```yaml
# .github/workflows/astro.yml
name: Deploy Astro site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "18"
          cache: npm
      - run: npm ci
      - run: npx playwright install --with-deps chromium
      - run: npm run build
      - run: touch ./dist/.nojekyll
      - uses: actions/upload-pages-artifact@v3
        with:
          path: ./dist

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - uses: actions/deploy-pages@v4
        id: deployment
```

### WordPress (Managed Host)

**Recommended:** Kinsta or WPEngine

**wp-config.php additions:**
```php
define('WP_ENVIRONMENT_TYPE', 'production');
define('GRAPHQL_JWT_AUTH_SECRET_KEY', '<secure-key>');
define('DISALLOW_FILE_EDIT', true);
define('HEADLESS_MODE_CLIENT_URL', 'https://mycosci.com');
```

---

## 16. Contributing

### Before Committing

1. **Build Check:** Run `npm run build` - must pass without errors
2. **Format:** Run `npx prettier --write` on changed files
3. **Avoid:** Do not edit `dist/` or `node_modules/`

### Commit Style

Use concise `type: description` format:

```
feat: add species page for Amanita muscaria
fix: correct taxonomy hierarchy for Cantharellus
docs: update cultivation guide for oyster mushrooms
refactor: simplify species template structure
```

### Pull Request Guidelines

1. Create feature branch from `main`
2. Make changes with clear commits
3. Run build locally to verify
4. Open PR with:
   - Summary of changes
   - Build result confirmation
   - Screenshots if UI changes

### Content Contributions

For new species entries:
1. Copy species template from Section 8
2. Fill all fields using data entry guide (Section 9)
3. Place in correct taxonomy path
4. Verify build passes
5. Submit PR

---

## Appendix: Quick Reference

### Edibility Badge Mapping

| Status | Variant | Color |
|--------|---------|-------|
| edible | success | green |
| inedible | default | gray |
| toxic | danger | red |
| psychoactive | tip | purple |
| unknown | note | blue |

### Sample ID Format

`MYCO-YYYY-NNNNN`
- `MYCO` - Prefix
- `YYYY` - Year
- `NNNNN` - 5-digit sequence number

Example: `MYCO-2026-00042`

### Storage Position Format

`D#-R#-B#-P##`
- `D#` - Dewar number
- `R#` - Rack number
- `B#` - Box number
- `P##` - Position (1-81 for 9x9 grid)

Example: `D1-R3-B2-P15`

### API Endpoints Summary

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/graphql` | POST | GraphQL API |
| `/wp-json/jwt-auth/v1/token` | POST | Get JWT token |
| `/wp-json/jwt-auth/v1/token/refresh` | POST | Refresh token |
| `/wp-json/wc/v3/products` | GET | WooCommerce products |
| `/wp-json/wc/v3/orders` | POST | Create order |

---

*This document serves as the single source of truth for the MycoSci project. Keep it updated as the project evolves.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MycoSci)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/MycoSci)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
