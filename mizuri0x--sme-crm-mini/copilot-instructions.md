## sme-crm-mini

> Lightweight SQLite CRM for small businesses. Manage clients, track deals,

# sme-crm-mini

Lightweight SQLite CRM for small businesses. Manage clients, track deals,
log timestamped notes, set follow-up reminders, and visualise your sales
pipeline — all from natural language in Agent Zero.

**Price:** 45 A0T | **Category:** business | **Author:** bankskills-dev

---

## Quick Start

The plugin uses a local SQLite database at `~/.sme-crm/crm.db`.
All commands output JSON for Agent Zero integration.
Run directly for human-readable ASCII tables.

```bash
bash run.sh add-client --name "Jan Kowalski" --company "Kowalski Sp. z o.o." \
  --email jan@kowalski.pl --deal-value 15000 --stage prospect
```

---

## Agent Zero Natural Language Examples

### Adding Clients

> **Polish:** "Dodaj klienta: Jan Kowalski, firma Kowalski Sp. z o.o., email jan@kowalski.pl, telefon 600-100-200, wartość dealu 15000 PLN, etap: prospect"

> **English:** "Add client Jan Kowalski from Kowalski Ltd, email jan@kowalski.pl, deal value 15000 PLN, stage prospect"

```bash
bash run.sh add-client \
  --name "Jan Kowalski" \
  --company "Kowalski Sp. z o.o." \
  --email "jan@kowalski.pl" \
  --phone "600-100-200" \
  --deal-value 15000 \
  --stage prospect
```

### Showing the Pipeline

> **Polish:** "Pokaż pipeline sprzedażowy"

> **English:** "Show me the sales pipeline"

```bash
bash run.sh pipeline
```

Outputs a Kanban-style ASCII board:
```
  +----------------------------------------------------+
  |       SME-CRM-MINI  //  SALES PIPELINE            |
  +----------------------------------------------------+
  | Clients: 3     Pipeline value: 45,000.00 PLN       |
  +----------------------------------------------------+
  [?] LEAD          ####        1 clients     10,000.00 PLN
  [~] PROSPECT      ########    2 clients     35,000.00 PLN
  [*] NEGOTIATION               0 clients             -
  [V] CLOSED                    0 clients             -
  [X] LOST                      0 clients             -
  +----------------------------------------------------+
```

### Adding Notes

> **Polish:** "Dodaj notatkę do Jana Kowalskiego: zainteresowany automatyzacją AI, prosi o demo w przyszłym tygodniu"

> **English:** "Add a note to Jan Kowalski: interested in AI automation, wants demo next week"

```bash
bash run.sh note \
  --client "Jan Kowalski" \
  --text "Zainteresowany automatyzacja AI, prosi o demo w przyszlym tygodniu"
```

### Setting Follow-ups

> **Polish:** "Ustaw follow-up z Kowalskim na 15 marca 2026, przypomnienie: wysłać ofertę"

> **English:** "Set a follow-up with Kowalski for March 15, reminder: send proposal"

```bash
bash run.sh follow-up \
  --client "Kowalski" \
  --date "2026-03-15" \
  --reminder "Wyslac oferte handlowa"
```

### Listing & Filtering

> "List all prospects" / "Pokaż wszystkich prospektów"

```bash
bash run.sh list --stage prospect
bash run.sh list
```

### Searching

> "Search for Kowalski" / "Znajdź klientów z branży IT"

```bash
bash run.sh search "Kowalski"
bash run.sh search "IT"
```

### Exporting to CSV

> **Polish:** "Wyeksportuj wszystkich klientów do CSV"

> **English:** "Export all clients to a CSV file"

```bash
bash run.sh export
bash run.sh export --output /tmp/clients_march_2026.csv
```

---

## Commands Reference

| Command | Key Arguments | Description |
|---------|--------------|-------------|
| `add-client` | `--name` (req), `--company`, `--email`, `--phone`, `--source`, `--deal-value`, `--stage` | Add new client |
| `list` | `--stage` (filter) | List all clients |
| `note` | `--client` (req), `--text` (req) | Add timestamped note |
| `follow-up` | `--client` (req), `--date` (req), `--reminder` | Schedule follow-up |
| `pipeline` | — | Kanban pipeline view |
| `search` | `query` | Full-text search |
| `export` | `--output` | CSV export |

## Pipeline Stages

| Stage | Icon | Description |
|-------|------|-------------|
| `lead` | `[?]` | Initial contact, unqualified |
| `prospect` | `[~]` | Qualified, in conversation |
| `negotiation` | `[*]` | Active deal negotiation |
| `closed` | `[V]` | Won deal |
| `lost` | `[X]` | Lost deal |

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `CRM_DB_PATH` | `~/.sme-crm/crm.db` | SQLite database path |

---

*sme-crm-mini v0.1.0 — bankskills-dev — MIT License*

---
> Source: [Mizuri0x/sme-crm-mini](https://github.com/Mizuri0x/sme-crm-mini) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
