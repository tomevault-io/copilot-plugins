## litbridge-backend

> Agent C: PRISMA deduplication engine — DOI/PMID hard match, fuzzy title, Golden Records


# Deduplication Engine (Agent C)

## Strategy: Two-Phase Dedup

### Phase 1 — Hard Match (exact identifiers)
1. **DOI match**: Normalize DOIs (lowercase, strip `https://doi.org/` prefix) → exact match
2. **PMID match**: If no DOI, match on PMID string equality
3. Records with matching DOI or PMID are the same publication — merge immediately

### Phase 2 — Soft Match (fuzzy, for records lacking identifiers)
1. **Title similarity**: Use `rapidfuzz.fuzz.token_sort_ratio` — threshold >= 90
2. **Year guard**: Titles must match AND publication year must match (or be within 1 year)
3. **Author overlap**: Optional tiebreaker — at least 1 common author surname

```python
from rapidfuzz import fuzz

def is_fuzzy_duplicate(a: RawRecord, b: RawRecord) -> bool:
    title_score = fuzz.token_sort_ratio(
        a.title.lower().strip(),
        b.title.lower().strip(),
    )
    if title_score < 90:
        return False
    if a.year and b.year and abs(a.year - b.year) > 1:
        return False
    return True
```

## Golden Record Construction

When duplicates are found, merge into a single canonical record:

```python
def build_golden_record(cluster: list[RawRecord]) -> UnifiedRecord:
    """Pick the richest metadata from each source in the cluster."""
    best = max(cluster, key=lambda r: _completeness_score(r))
    return UnifiedRecord(
        id=generate_stable_id(cluster),
        title=best.title,
        authors=best.authors or _first_non_empty(cluster, "authors"),
        doi=_first_non_empty(cluster, "doi"),
        pmid=_first_non_empty(cluster, "pmid"),
        abstract=_longest(cluster, "abstract"),
        source=best.source,
        sources_found_in=[r.source for r in cluster],
        year=best.year,
        duplicate_cluster_id=generate_stable_id(cluster),
    )
```

## Performance

- In-memory dedup for speed (typical result sets < 10K records)
- Index DOIs + PMIDs in `dict` for O(1) hard match lookups
- Fuzzy matching only on the remaining unmatched records
- Use `rapidfuzz` (C++ backend) — never pure-Python Levenshtein

## PRISMA Counters

Track at each stage for the PRISMA flow:

```python
@dataclass
class DedupStats:
    identified: int       # total raw records from all sources
    duplicates_removed: int
    after_dedup: int      # identified - duplicates_removed
    sources_breakdown: dict[str, int]  # count per source
```

## Rules

1. **Normalize before comparing** — lowercase titles, strip DOI prefixes, trim whitespace
2. **DOI is king** — if DOIs match, records are duplicates regardless of title
3. **Never lose provenance** — Golden Record tracks `sources_found_in` list
4. **rapidfuzz >= 3.0** — use `token_sort_ratio` for title comparison
5. **Log dedup stats** — identified, removed, after_dedup for every search
6. **Dedup is deterministic** — same input always produces same output (no randomness)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dawit-Getachew) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
