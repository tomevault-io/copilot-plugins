## channel-atlas

> Operating contract for AI agents (Claude, Codex, Cursor) working in

# AGENTS.md — channel-atlas

Operating contract for AI agents (Claude, Codex, Cursor) working in
this repo. The conventions match the rest of the AthenaTheOwl
portfolio. An agent already trained on supplier-risk-rag-agent or
chip-supply-chain-map will recognize the shape.

## What this repo is

A citation-faithful catalog of hyperscaler off-balance-sheet AI-infra
financing. The output is a quarterly published report plus a
machine-readable counterparty graph, not a live SaaS. Every edge in
the graph cites a public filing.

## Roles you may see in tasks

| Role | What they do |
|---|---|
| `edgar-extractor` | Pulls 8-K / 10-K / S-1 disclosures; resolves SPV references to filer entities |
| `lp-disclosure-walker` | Reads state pension, CalPERS, CPP, university endowment LP reports |
| `spv-registry-resolver` | Cross-checks Cayman, Delaware, Luxembourg SPV registrations |
| `graph-builder` | Composes the counterparty graph; runs entity-resolution and dedup |
| `report-renderer` | Produces the quarterly markdown / PDF report |
| `citation-auditor` | Gate: every node and edge has a verifiable public source |

These roles exist in the spec ledger; not all are implemented in v0.

## Voice constraints

- Plain assertions. No marketing words. The banned set lands in
  `scripts/voice_lint.py::BANNED_FAIL` in spec 0002.
- No antithetical reversals as a structural device.
- Numbers cite sources. A claim without a filing URL is a bug.

## Gates (will land in spec 0002)

Before merging, every contributor — human or agent — will run:

```bash
uv run pytest
python scripts/voice_lint.py
python scripts/validate_schemas.py
python scripts/check_citation_faithfulness.py
```

The citation-faithfulness gate fails when an edge in the graph lacks a
URL anchor into a public filing.

## Out of scope

- Real-time deal-flow alerts. This is a quarterly publication, not a
  Bloomberg terminal.
- Private deal sourcing or pricing. Public-filings-only by charter.
- Equity-side analyst calls or trade recommendations.
- Hosted SaaS dashboards. The artifact is a report plus a graph file.

---
> Source: [AthenaTheOwl/channel-atlas](https://github.com/AthenaTheOwl/channel-atlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
