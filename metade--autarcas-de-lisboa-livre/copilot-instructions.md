## autarcas-de-lisboa-livre

> Jekyll website for the Livre party's elected officials (autarcas) in Lisbon. Hosted on GitHub Pages, with Sveltia CMS for content editing.

# Autarcas de Lisboa – Livre

Jekyll website for the Livre party's elected officials (autarcas) in Lisbon. Hosted on GitHub Pages, with Sveltia CMS for content editing.

Deployed at: `https://autarcas-lisboa-livre.metade.org`

## Stack

- **Jekyll 4.3** — static site generator
- **Tailwind CSS 3** — utility-first styling, built via npm CLI before Jekyll
- **Sveltia CMS** — headless CMS loaded from CDN at `/admin/`, GitHub backend
- **GitHub Pages** — hosting via GitHub Actions (not legacy branch deploy)
- **Cloudflare Worker** — OAuth proxy for Sveltia CMS GitHub authentication, deployed at `https://autarcas-de-lisboa-cms-auth.metade.workers.dev`
- **`_plugins/date_filter.rb`** — Ruby plugin providing `date_pt` Liquid filter (Portuguese date formatting)
- **`_data/pt.yml`** — Portuguese month names used by the date filter

## Local Development

```bash
npm install          # install Tailwind
bundle install       # install Jekyll gems
npm run watch:css    # rebuild CSS on changes
bundle exec jekyll serve  # serve at http://127.0.0.1:4000
```

For a one-shot build: `npm run build:css && bundle exec jekyll build`

## Project Structure

```
_autarcas/       One .md per elected official (16 files)
_juntas/         One .md per parish assembly (9 files — Assembleias de Freguesia only)
_propostas/      Per-organ subfolders: _propostas/{organ}/{year}-{slug}.md + co-located PDFs
_tags/           Controlled tag vocabulary (output: false); each file has a `nome` field
_locations/      Controlled location vocabulary (output: false); each file has `nome` + `junta` fields
_pages/          Static pages (registered as a Jekyll collection so Jekyll outputs them)
_layouts/        default, autarca, junta, proposta, propostas_organ, page
_includes/       head, nav, footer, autarca-card, proposta-card
_plugins/        date_filter.rb — date_pt Liquid filter for Portuguese dates
_data/           pt.yml — Portuguese month names
assets/css/      main.css (Tailwind input) → main.min.css (compiled, gitignored)
assets/js/       filter.js — vanilla JS client-side filtering for proposals
admin/           Sveltia CMS entry (index.html) and config (config.yml)
cloudflare-worker/  OAuth proxy source (src/index.js, wrangler.toml)
```

> **Tailwind note:** After adding new utility classes to templates, rerun `npm run build:css` so they are included in the compiled output. Classes not present at build time will be silently dropped.

## Site Structure

| URL | Source |
|-----|--------|
| `/` | `index.md` |
| `/camara-municipal/` | `_pages/camara-municipal.md` (uses `junta` layout) |
| `/assembleia-municipal/` | `_pages/assembleia-municipal.md` (uses `junta` layout) |
| `/juntas/` | `_pages/juntas.md` — lists the 9 parish assemblies |
| `/juntas/:slug/` | `_juntas/*.md` |
| `/autarcas/` | `_pages/autarcas.md` |
| `/autarcas/:slug/` | `_autarcas/*.md` |
| `/propostas/` | `_pages/propostas.md` — filterable listing |
| `/propostas/:organ/` | `_pages/propostas-{organ}.md` — per-organ listing (uses `propostas_organ` layout) |
| `/propostas/:organ/:slug/` | `_propostas/{organ}/*.md` |
| `/sobre/` | `_pages/sobre.md` |
| `/admin/` | Sveltia CMS interface |

**Navigation order:** Câmara Municipal → Assembleia Municipal → Juntas de Freguesia → Autarcas → Propostas

## Content Model

### Autarcas (`_autarcas/*.md`)
One file per person, regardless of how many roles they hold. A person with roles in multiple organs (e.g. João Monteiro) has a single profile page listing all roles.

Key fields:
- `genero` — `m | f | n`, drives grammatical gender in Portuguese role display (o/a convention)
- `juntas` — flat list of junta slugs for Liquid filtering (`where_exp: "a.juntas contains slug"`)
- `cargos` — structured list of cargo objects, displayed on the profile page

Cargo object fields:
- `cargo`, `orgao`, `junta` — required
- `ausente_temporariamente` — boolean; member is temporarily suspended
- `temporario` — boolean; this is a temporary substitution role
- `substitui` — slug of the member being substituted (set on the substitute)
- `substituido_por` — slug of the substitute (set on the suspended member)

Example (substitute member):
```yaml
cargos:
  - cargo: Membro de Assembleia
    orgao: Assembleia de Freguesia de Arroios
    junta: arroios
    temporario: true
    substitui: patricia-robalo
    substituido_por: ''
```

Example (suspended member):
```yaml
cargos:
  - cargo: Membro de Assembleia
    orgao: Assembleia de Freguesia de Arroios
    junta: arroios
    ausente_temporariamente: true
    substituido_por: patrick-sinclair
```

Junta pages render suspended members in a separate faded row below active members.

### Juntas (`_juntas/*.md`)
Only the 9 parish-level assemblies. Câmara Municipal and Assembleia Municipal are standalone pages in `_pages/`, not in this collection.

Key fields: `nome`, `slug`, `tipo`, `descricao`, `foto_junta`

Social link fields (all optional, empty string if unused): `website_oficial`, `facebook`, `x`, `instagram`, `youtube`

### Autarca social fields
Optional contact/social fields on `_autarcas/*.md`: `contacto_email`, `contacto_twitter`, `contacto_instagram`, `contacto_linkedin` (full URL), `contacto_bluesky` (handle only), `contacto_facebook` (full URL). All default to empty string.

### Propostas (`_propostas/{organ}/*.md`)
Files live in per-organ subfolders (e.g. `_propostas/arroios/2026-foo.md`). PDFs are co-located alongside the `.md` file and automatically copied to the output. The `junta` frontmatter field is auto-set by Jekyll defaults based on folder path.

Key fields:
- `junta` — single slug (for `where` filtering by organ)
- `autarcas` — list of autarca slugs (for cross-linking and filtering)
- `estado` — `Em análise | Aprovada | Rejeitada | Retirada`
- `tipo` — `Proposta | Moção | Requerimento | Voto`
- `tags` — list of tag names; values must match the `nome` field of an entry in `_tags/`
- `localizacoes` — list of location names; values must match the `nome` field of an entry in `_locations/`

The CMS is configured with 11 per-organ collections (Câmara Municipal, Assembleia Municipal, + 9 junta-level organs). All slug/relation fields use CMS relation widgets — editors pick from dropdowns rather than typing slugs by hand. The `localizacoes` relation widget is filtered by junta in each per-organ collection.

## Liquid Relationships

All relationships are resolved at build time via Liquid filters — no plugins required:

```liquid
{% comment %} Autarcas in a given organ {% endcomment %}
{% assign eleitos = site.autarcas | where_exp: "a", "a.juntas contains page.slug" %}

{% comment %} Proposals in a given organ {% endcomment %}
{% assign propostas = site.propostas | where: "junta", page.slug %}

{% comment %} Proposals by a given autarca {% endcomment %}
{% assign propostas = site.propostas | where_exp: "p", "p.autarcas contains page.slug" %}
```

## Sveltia CMS Setup

Fully configured and deployed:

- `repo: metade/autarcas-de-lisboa-livre`
- `base_url: https://autarcas-de-lisboa-cms-auth.metade.workers.dev`
- GitHub App callback: `https://autarcas-de-lisboa-cms-auth.metade.workers.dev/callback`
- Worker secrets (`GITHUB_CLIENT_ID`, `GITHUB_CLIENT_SECRET`) stored in Wrangler, not committed
- Deploy the worker: `cd cloudflare-worker && wrangler deploy`
- Grant write access to the GitHub repo for anyone who needs CMS access

## GitHub Pages Setup

Fully configured and deployed:

- Workflow at `.github/workflows/pages.yml` builds and deploys on push to `main`
- Uses `--baseurl /autarcas-de-lisboa-livre` for the GitHub Pages path prefix
- In repo Settings → Pages, source must be set to **GitHub Actions**

## Elected Officials (2025 mandate)

16 officials across 11 organs: 1 at Câmara Municipal, 2 at Assembleia Municipal, and 13 across 9 parish assemblies (Avenidas Novas, Santo António, Alvalade, São Domingos de Benfica, Lumiar, Areeiro, Arroios, Parque das Nações, Penha de França). See `_autarcas/` for individual profiles.

Notable model cases:
- One person can hold multiple roles — João Monteiro has roles in both Assembleia Municipal and Penha de França, with a single profile at `/autarcas/joao-monteiro/`
- Arroios currently has an active substitution: one member is `ausente_temporariamente`, replaced by a `temporario` substitute

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/metade) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
