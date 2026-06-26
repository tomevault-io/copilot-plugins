## sliderule-schema-server

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A deploy pipeline for a static JSON-only CloudFront distribution that serves the SlideRule schema endpoints at `schema.testsliderule.org` (test) and `schema.slideruleearth.io` (prod, future). **No Lambda, no Lambda@Edge, no CloudFront Function** — CloudFront strips the leading `/` from the request path and looks up that exact key in an S3 bucket. URL path, S3 key, and the on-disk tree under [schema-endpoints/merged/source/](schema-endpoints/merged/source/) are identical 1:1.

This is also the source of truth for a packaged Claude skill at [skills/sliderule-schema/](skills/sliderule-schema/) — a thin HTTPS client that fetches from the deployed distribution.

## The three-tier content model (load-bearing)

```
schema-endpoints/
├── authored/     humans edit here (taxonomy, couplings, index)
├── generated/    tool-emitted (hand-maintained for now)
├── merged/       build artifact — COMMITTED TO GIT, never hand-edited
└── merge.py      authored/ + generated/ → merged/
```

`merged/` is a committed build artifact. This is deliberate and unusual — the rationale is "paired-diff review": every source edit under `authored/` or `generated/` is committed **alongside** the resulting `merged/` diff, so reviewers see the exact bytes going to S3 next to the edit that caused them.

**Workflow for any edit under `authored/` or `generated/`:**

```bash
# edit files
python3 schema-endpoints/merge.py   # refresh merged/
make verify                         # asserts merged/ matches merge.py output
git add schema-endpoints/           # include BOTH source edit AND merged/
git commit
```

Never commit a source edit without its paired `merged/` diff — `make verify` (and CI) will fail. Never hand-edit files under `merged/`.

`merge.py` has preflight validators that fail loudly before touching `merged/`: advertised `param_count` matches reality, URL labels agree with targets, params in `structure.json` exist in `generated/params.json`, no orphan params, no duplicates across groups. Fix the source, not the validation.

## Common commands

```bash
# Local (no AWS needed):
make verify                       # merge.py drift check + terraform fmt check
python3 schema-endpoints/merge.py # refresh merged/ manually
make build                        # runs merge.py, stages merged/ → build/
make clean                        # remove build/
make tf-fmt                       # format terraform in place

# Deploys (AWS creds required; per-env wrappers carry DOMAIN/S3_BUCKET/DOMAIN_APEX):
make deploy-to-testsliderule      # terraform-apply + live-update (first time)
make live-update-testsliderule    # content-only: verify + build + s3 sync + invalidate
make smoketest-testsliderule      # curl the endpoints, check status/Content-Type/CORS
make destroy-testsliderule        # tear down

# Per-env -slideruleearth wrappers exist for everything above.

# Observability:
make requests-testsliderule       # CloudFront request count, 1h buckets, last 24h

# Skill packaging (output gitignored, zipped into /tmp_skill_for_export/):
make package-skill-schema
```

`DISTRIBUTION_ID` is auto-resolved from the domain alias via `aws cloudfront list-distributions`. `terraform` uses one workspace per domain.

## Notable constraints

- **Python 3.11+** required for `merge.py` (modern type hints: `list[str]`, `dict | None`, `Path.is_file()`).
- **Terraform 1.5+**. `terraform/.terraform.lock.hcl` is committed (per HashiCorp recommendation) to pin provider SHAs across the team; `terraform init -lockfile=readonly` in CI enforces this.
- **No Lambda anywhere** in this architecture — S3 + CloudFront only. Do not propose Lambda-based solutions (those belong in the sliderule-search-server sibling repo).
- **AWS creds aren't needed** for `make verify`, `make build`, or running `merge.py`. They're only needed for `deploy-*`, `destroy-*`, `live-update-*`, `smoketest-*`, and `requests-*`.
- **404 behavior is content, not code.** Unpublished paths (e.g. `swot.json`, `cre.json`) 404 via CloudFront `custom_error_response` pointing at `/errors/not-found.json`. The body of that file is in `authored/errors/not-found.json` and is deployed alongside the schema tree.

## The packaged skill ([skills/sliderule-schema/](skills/sliderule-schema/))

Laid out per Anthropic's progressive-disclosure guidance:

```
skills/sliderule-schema/
├── SKILL.md                       # under 200 lines; cap per best-practices
├── requirements.txt
├── scripts/schema.py              # thin HTTPS client; no business logic
├── reference/core-params.md       # exhaustive core-params list + override rules
└── evals/                         # rubric files matching Anthropic's example JSON shape
    ├── README.md
    └── 0[1-6]-*.json
```

When editing SKILL.md, prefer moving exhaustive data to `reference/` over inlining. When changing `scripts/schema.py` behavior or URL layout, update both SKILL.md's "Agent instructions" section and the relevant eval's `expected_behavior` entries — evals are grounded against the live schema and go stale if the schema changes.

## CI / branch protection

[.github/workflows/ci.yml](.github/workflows/ci.yml) runs on every PR/push to main and mirrors `make verify` exactly: merge.py drift check, `terraform fmt -check`, `terraform init -backend=false && terraform validate -lockfile=readonly`, deterministic re-run of `merge.py`, and `make package-skill-schema` as a packaging sanity check. No AWS creds are configured in CI — everything is read-only against the repo.

## Where things come from (long-term)

Some `generated/` files are hand-maintained today but have a long-term source:

- `<domain>/params.json` → sliderule server's `/source/defaults` endpoint
- `icesat2/fields/<sel>.json`, `gedi/fields/<sel>.json` → `scripts/enumerate_h5_fields.py` (adopted from the sliderule repo in Apr 2026; ATL24 and GEDI products added at that time)
- `<domain>/output/<api>.json` → `sliderule/scripts/test_server_generated_schema.sh`

See [README.md](README.md) "Regenerating the source JSON" for the copy-in commands.

## Sibling repos worth knowing

- **sliderule-web-client** — the Makefile's per-env wrapper pattern (`deploy-to-testsliderule`, `deploy-to-slideruleearth`) deliberately mirrors that repo's conventions.
- **sliderule-search-server** — Lambda-backed docsearch service. Its `make requests` observability pattern was ported here; its Lambda-specific targets (`logs`, `errors`, `invocations`, `cost-estimate`) don't apply here since there's no Lambda.
- **sliderule** — the main SlideRule repo; source of the `generated/` files listed above.

---
> Source: [SlideRuleEarth/sliderule-schema-server](https://github.com/SlideRuleEarth/sliderule-schema-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
