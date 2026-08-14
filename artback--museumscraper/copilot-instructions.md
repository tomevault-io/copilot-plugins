## museumscraper

> One Go binary, `museum`, with six subcommands that never call each other. See

# Working on the museum catalogue

One Go binary, `museum`, with six subcommands that never call each other. See
README.md for what each one does and why the architecture is shaped this way.
This file is only about running it.

## Tests

```bash
go test -short ./...        # offline only — use this by default
go test -race -short ./...  # before committing anything concurrent
go test ./...               # also hits Wikidata, Wikipedia, Overpass, Nominatim
```

Prefer `-short`. The full run makes real requests to rate-limited public
services, and Nominatim's usage policy is not a suggestion.

## Running locally

```bash
docker compose up -d                    # postgres+postgis, minio, kafka, api, enricher
docker compose run --rm jobs crawl      # batch jobs go through the "jobs" service
docker compose run --rm jobs reindex
docker compose run --rm jobs verify -samples 3
```

The API is then on <http://localhost:8090>, the map on `/map`.

Batch jobs share one `jobs` service on purpose: `docker compose run` replaces a
service's command outright, so giving each job its own service with a baked-in
subcommand meant the subcommand vanished the moment you passed a flag.

## Running in production

It runs on the homelab Nomad cluster — a single Raspberry Pi, `100.116.81.88`,
reachable over Tailscale.

| | |
|---|---|
| Map | <http://100.116.81.88/map> |
| API | `http://100.116.81.88:8091` (fabio also routes `/map` and `/v1`) |
| Job spec | `packs/museum` in [artback/iac_jonathan](https://github.com/artback/iac_jonathan) |
| Deploy | push to `main` — see `.github/workflows/README.md` |

The job spec deliberately lives in the IaC repository alongside the sixteen
other services on the same Pi; this repository holds only the workflow that
builds the image and triggers a deploy.

To deploy by hand instead:

```bash
cd ~/Code/iac_jonathan
nomad-pack run packs/museum -f vars/museum-kalmar.pkrvars.hcl
```

## Four things that will bite you

**The Pi is arm64.** Any image the job references must be too. `postgis/postgis`
publishes amd64 only and simply will not run there; `imresamu/postgis` is the
multi-arch build of the same Dockerfile. A wrong-architecture image fails as an
opaque `exec format error` long after the build went green.

**Seeding must not raise bucket events.** The enricher does not check whether a
record has already been enriched — it geocodes every museum an event names, then
makes a second Nominatim call for the place details. Loading the existing
345,000 objects into a bucket with the notification attached queues roughly
690,000 requests against a service that allows one per second. Use
`seed_mode = true` for bulk loads; `packs/museum/scripts/migrate-data.sh`
refuses to run without it.

**Ports on the Pi are contested and not all discoverable.** 8090 is beszel, 9000
is mealie, 5432 is the shared postgres, 9100 is node-exporter. That last one has
no pack in the IaC repository, so grepping the job files does not find it —
taking it put node-exporter into a restart loop and blinded Prometheus. Check
the running cluster, not just the repo.

**Tasks in a Nomad group share a network namespace.** A task reaching a sibling
through the host's published port hairpins back into its own namespace and never
connects. Use `localhost` and the container port for siblings; the host IP and
the static port only for other groups.

---
> Source: [artback/museumscraper](https://github.com/artback/museumscraper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
