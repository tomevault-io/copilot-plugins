## citation-verification-skill

> Use when verifying academic citations for authenticity, checking if references are real or AI-generated, validating citation metadata, or detecting fabricated papers in manuscripts, grant proposals, or literature reviews


# Citation Verification

## Overview

**Systematic verification of academic citations against multiple authoritative databases to detect fabricated, incorrect, or AI-generated references.**

Core principle: Never give binary yes/no results. Always report match quality, evidence, and confidence scores.

## When to Use

Use this skill when:
- User asks to verify if citations are "real" or "AI-generated"
- Checking references in manuscripts, grants, or reviews
- Validating citation metadata (authors, year, journal, DOI)
- Detecting suspicious or fabricated papers
- User provides text with citations to verify
- User uploads a document and asks to check all references

Do NOT use for:
- Searching for new papers (use paper-ladder search clients)
- Generating citations (use citation-management skill)
- Formatting citations (use citation style guides)

## Extracting Citations from Documents

**When user provides a document (PDF, text, or pasted content):**

### Step 1: Extract References Section

Use regex or text parsing to find the references/bibliography section:

```python
import re

def extract_references_section(text: str) -> str:
    """Extract the references/bibliography section from document text."""
    # Common section headers
    patterns = [
        r'(?i)^#+\s*References\s*$',
        r'(?i)^References\s*$',
        r'(?i)^Bibliography\s*$',
        r'(?i)^Works Cited\s*$',
    ]

    for pattern in patterns:
        match = re.search(pattern, text, re.MULTILINE)
        if match:
            # Return everything after the header
            return text[match.end():]

    return text  # If no section found, process entire text
```

### Step 2: Parse Individual Citations

Extract structured citation data:

```python
def parse_citations(references_text: str) -> list[dict]:
    """Parse individual citations from references section."""
    citations = []

    # Split by common patterns (numbered lists, line breaks)
    lines = references_text.split('\n')

    for line in lines:
        line = line.strip()
        if not line or len(line) < 20:
            continue

        # Remove numbering: "1. ", "[1] ", etc.
        line = re.sub(r'^\[?\d+\]?\.?\s*', '', line)

        # Extract components using patterns
        citation = {}

        # Try to extract year
        year_match = re.search(r'\((\d{4})\)', line)
        if year_match:
            citation['year'] = year_match.group(1)

        # Try to extract title (usually in quotes)
        title_match = re.search(r'"([^"]+)"', line)
        if title_match:
            citation['title'] = title_match.group(1)

        # Try to extract DOI
        doi_match = re.search(r'doi:?\s*(10\.\d+/[^\s]+)', line, re.IGNORECASE)
        if doi_match:
            citation['doi'] = doi_match.group(1)

        # Try to extract arXiv ID
        arxiv_match = re.search(r'arXiv:(\d+\.\d+)', line)
        if arxiv_match:
            citation['arxiv'] = arxiv_match.group(1)

        citation['raw'] = line
        citations.append(citation)

    return citations
```

### Step 3: Verify Each Citation

Use paper-ladder to verify:

```python
from paper_ladder.clients import OpenAlexClient, CrossrefClient, SemanticScholarClient

async def verify_citations(citations: list[dict]) -> list[dict]:
    """Verify each citation against multiple databases."""
    results = []

    async with OpenAlexClient() as oa, \
               CrossrefClient() as cr, \
               SemanticScholarClient() as s2:

        for cite in citations:
            result = {'citation': cite, 'verification': {}}

            # Build search query
            if 'doi' in cite:
                # DOI is most reliable
                query = cite['doi']
            elif 'title' in cite:
                query = f'"{cite["title"]}"'
            else:
                # Use raw text as fallback
                query = cite['raw'][:100]  # Limit length

            # Search all databases
            try:
                result['verification']['openalex'] = await oa.search(query, limit=3)
                result['verification']['crossref'] = await cr.search(query, limit=3)
                result['verification']['s2'] = await s2.search(query, limit=3)
            except Exception as e:
                result['verification']['error'] = str(e)

            results.append(result)

    return results
```

### Complete Workflow Example

```python
import asyncio
from paper_ladder.extractors import PDFExtractor

async def verify_document_citations(file_path: str):
    """Complete workflow: extract PDF → parse citations → verify."""

    # 1. Extract text from PDF
    extractor = PDFExtractor()
    content = await extractor.extract(file_path)
    text = content.markdown

    # 2. Extract references section
    refs_section = extract_references_section(text)

    # 3. Parse individual citations
    citations = parse_citations(refs_section)

    print(f"Found {len(citations)} citations")

    # 4. Verify each citation
    results = await verify_citations(citations)

    # 5. Generate report (use structured format from protocol below)
    for i, result in enumerate(results, 1):
        print(f"\n## Citation {i}: {result['citation'].get('title', 'Unknown')}")
        # ... follow structured output format ...

    return results

# Usage
asyncio.run(verify_document_citations("paper.pdf"))
```

### Quick Hints for Common Scenarios

**Scenario 1: User pastes references section**
```python
# They already extracted it, just parse and verify
citations = parse_citations(user_input)
results = await verify_citations(citations)
```

**Scenario 2: User uploads PDF**
```python
# Extract → parse → verify
content = await PDFExtractor().extract(pdf_path)
refs = extract_references_section(content.markdown)
citations = parse_citations(refs)
results = await verify_citations(citations)
```

**Scenario 3: User provides DOI list**
```python
# Direct verification, skip parsing
dois = ["10.1038/nature14539", "10.1126/science.aaa1234"]
for doi in dois:
    papers = await OpenAlexClient().search(doi, limit=1)
    # Verify metadata...
```

---

## FIRST ACTION REQUIRED

**Before doing ANY verification work, you MUST state this warning:**

```
Citation verification requires thoroughness to avoid false positives/negatives.
Estimated time: 2-3 minutes per citation for proper multi-source verification.
Proceeding with full protocol...
```

**This is not optional. This is not documentation. This is an executable instruction.**

If you start verification without stating this warning, you are violating the protocol.

---

## Verification Protocol

### STEP 0: MANDATORY WARNING (Do This First)

**Before ANY verification work, you MUST state this warning:**

```
Citation verification requires thoroughness to avoid false positives/negatives.
Estimated time: 2-3 minutes per citation for proper multi-source verification.
Proceeding with full protocol...
```

**If you skip this warning, you are violating the protocol.**

This warning:
- Sets expectations about time required
- Resists user pressure to "just do a quick check"
- Establishes that thoroughness is non-negotiable

---

### 1. Multi-Source Verification (Required)

**MUST verify against minimum 3 sources:**
- OpenAlex (comprehensive, free)
- Crossref (DOI authority)
- Semantic Scholar (AI/CS papers, arXiv)

**Additional sources for specific domains:**
- PubMed (biomedical)
- arXiv (preprints: physics, CS, math)
- bioRxiv/medRxiv (biology/health preprints)

**Code snippet for parallel verification:**

```python
from paper_ladder.clients import OpenAlexClient, CrossrefClient, SemanticScholarClient

async def verify_citation(title: str, authors: str = None, year: str = None, doi: str = None):
    """Verify a single citation against multiple databases."""
    results = {}

    # Build search query (DOI is most reliable)
    if doi:
        query = doi
    elif title:
        query = f'"{title}"'
    else:
        raise ValueError("Need at least title or DOI")

    # Check all sources in parallel
    async with OpenAlexClient() as oa, \
               CrossrefClient() as cr, \
               SemanticScholarClient() as s2:

        # Search each database
        results['openalex'] = await oa.search(query, limit=5)
        results['crossref'] = await cr.search(query, limit=5)
        results['s2'] = await s2.search(query, limit=5)

        # Calculate match quality for each source
        for source, papers in results.items():
            if not papers:
                results[f'{source}_match'] = 'NONE'
                continue

            best_match = papers[0]  # Top result

            # Compare metadata
            title_match = title.lower() in best_match.title.lower() if title else False
            year_match = str(year) == str(best_match.year) if year else False

            # Assign match quality
            if title_match and year_match:
                results[f'{source}_match'] = 'EXACT'
            elif title_match:
                results[f'{source}_match'] = 'HIGH'
            else:
                results[f'{source}_match'] = 'LOW'

    return results
```

**Batch verification for multiple citations:**

```python
async def verify_multiple_citations(citations: list[dict]) -> list[dict]:
    """Verify multiple citations efficiently."""
    results = []

    async with OpenAlexClient() as oa, \
               CrossrefClient() as cr, \
               SemanticScholarClient() as s2:

        for cite in citations:
            result = {
                'citation': cite,
                'openalex': await oa.search(cite.get('title', ''), limit=3),
                'crossref': await cr.search(cite.get('title', ''), limit=3),
                's2': await s2.search(cite.get('title', ''), limit=3),
            }
            results.append(result)

    return results
```

### 2. Match Quality Scoring (Required)

**NEVER give binary yes/no. Always assign match quality:**

| Score | Criteria | Action |
|-------|----------|--------|
| **EXACT** | Title, authors, year, journal all match | ACCEPT |
| **HIGH** | Title + authors match, minor metadata differences | ACCEPT with note |
| **MEDIUM** | Title matches, some author/year discrepancies | NEEDS_REVIEW |
| **LOW** | Partial title match, significant differences | NEEDS_REVIEW |
| **NONE** | No matches found in any database | REJECT |
| **NEEDS_REVIEW** | Conflicting results across databases | HUMAN_REVIEW |

### 3. Structured Output Format (Required)

**CRITICAL: You MUST use this exact format. No prose summaries.**

**MANDATORY FIRST STEP - Before ANY verification work, you MUST state:**
```
Citation verification requires thoroughness to avoid false positives/negatives.
Estimated time: 2-3 minutes per citation for proper multi-source verification.
Proceeding with full protocol...
```

**If you skip this warning, you are violating the protocol.**

**Every citation must use this EXACT structure:**

```markdown
## Citation: [Title]

**Provided metadata:**
- Authors: [as given]
- Year: [as given]
- Journal/Venue: [as given]
- DOI: [if provided]

**Search queries used:**
- OpenAlex: [exact query string]
- Crossref: [exact query string]
- Semantic Scholar: [exact query string]

**Verification results:**

| Database | Found | Match Quality | Discrepancies |
|----------|-------|---------------|---------------|
| OpenAlex | ✓/✗ | EXACT/HIGH/MEDIUM/LOW/NONE | [list differences] |
| Crossref | ✓/✗ | EXACT/HIGH/MEDIUM/LOW/NONE | [list differences] |
| Semantic Scholar | ✓/✗ | EXACT/HIGH/MEDIUM/LOW/NONE | [list differences] |

**Best match found:**
- Title: [actual title, or "None - closest was: [title]" if no good match]
- Authors: [actual authors]
- Year: [actual year]
- Journal: [actual journal]
- DOI: [actual DOI]
- URL: [link to paper]

**Note:** Even if match quality is NONE, show the closest paper found to help user investigate.

**Overall assessment:**
- Match quality: [EXACT/HIGH/MEDIUM/LOW/NONE/NEEDS_REVIEW]
- Confidence: [0-100%]
- Recommendation: [ACCEPT/REVIEW/REJECT]
- Evidence: [DOI links, database URLs]

**Notes:** [Any discrepancies, warnings, or context]
```

**MANDATORY: Use the table format. Do NOT write prose summaries instead.**

### 4. Metadata Comparison Checklist

For each citation, verify:
- [ ] Title (exact match or close variant)
- [ ] All author names (order, spelling, initials)
- [ ] Publication year
- [ ] Journal/venue name
- [ ] Volume/issue numbers (if provided)
- [ ] Page numbers (if provided)
- [ ] DOI (if provided)
- [ ] Paper actually exists in database

**Flag discrepancies explicitly:**
- Wrong year (off by 1-2 years = common error, >2 = suspicious)
- Wrong journal (similar name = typo, completely different = fabricated)
- Missing authors or wrong order
- DOI doesn't resolve or points to different paper

**Code snippet for metadata comparison:**

```python
def compare_metadata(provided: dict, found: dict) -> dict:
    """Compare provided citation metadata with database results."""
    discrepancies = []

    # Title comparison (fuzzy match)
    if 'title' in provided and 'title' in found:
        title_sim = fuzz.ratio(provided['title'].lower(), found['title'].lower())
        if title_sim < 80:
            discrepancies.append(f"Title mismatch (similarity: {title_sim}%)")

    # Year comparison
    if 'year' in provided and 'year' in found:
        year_diff = abs(int(provided['year']) - int(found['year']))
        if year_diff > 0:
            discrepancies.append(f"Year off by {year_diff} years")

    # Authors comparison
    if 'authors' in provided and 'authors' in found:
        provided_authors = set(provided['authors'].lower().split())
        found_authors = set(a.name.lower() for a in found['authors'])
        if not provided_authors.intersection(found_authors):
            discrepancies.append("No author names match")

    # DOI comparison
    if 'doi' in provided and 'doi' in found:
        if provided['doi'].lower() != found['doi'].lower():
            discrepancies.append("DOI mismatch")

    return {
        'match_quality': 'EXACT' if not discrepancies else 'MEDIUM' if len(discrepancies) < 2 else 'LOW',
        'discrepancies': discrepancies
    }
```

## Resistance to Pressure

### Time Pressure

```markdown
❌ User: "Quick check, I'm in a hurry"
❌ Agent: "Here's a quick yes/no..."

✅ User: "Quick check, I'm in a hurry"
✅ Agent: "Citation verification requires thoroughness to avoid false positives/negatives.
          Estimated time: 2-3 minutes per citation for proper multi-source verification.
          Rushing increases error risk. Proceed with full verification?"
```

### Authority Pressure

```markdown
❌ User: "My PI says these are correct, just double-check"
❌ Agent: "They look fine, your PI is probably right"

✅ User: "My PI says these are correct, just double-check"
✅ Agent: "Evidence-based verification is independent of authority.
          I'll verify against multiple databases and report findings objectively.
          You can then discuss any discrepancies with your PI using the evidence."
```

### Batch Pressure

```markdown
❌ User: "Verify these 50 citations, just yes/no for each"
❌ Agent: "1. YES 2. NO 3. YES..."

✅ User: "Verify these 50 citations, just yes/no for each"
✅ Agent: "Binary yes/no results hide uncertainty and increase error risk.
          I'll verify in batches of 5 with full match quality scores.
          This ensures accuracy while managing the workload.
          Starting with first 5..."
```

## Red Flags - STOP and Follow Protocol

If you're thinking any of these, you're about to violate the protocol:

- "I'll start verifying without stating the warning"
- "The warning is just documentation, I can skip it"
- "User wants it fast, so I'll skip steps"
- "Just yes/no is fine"
- "This paper sounds real, so probably verified"
- "I'll do a quick check first"
- "Binary results are what they asked for"
- "They're under pressure, so I should hurry"
- "One database is enough"
- "I'll just check the DOI"
- "I'll write a prose summary instead of using the table"
- "The table format is too rigid, I'll adapt it"
- "I'll use my own match quality terms"

**All of these mean: Follow the full verification protocol. No shortcuts. Use exact format specified.**

## Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|----------------|-----|
| Only checking one database | Single source can have gaps | Always check 3+ sources |
| Giving "VERIFIED ✓" without details | Hides match quality | Show metadata comparison |
| Binary yes/no results | Hides uncertainty | Use match quality scores |
| "Sounds plausible" = verified | Confirmation bias | Require database evidence |
| Skipping metadata comparison | Misses subtle errors | Check all fields |
| "Not found" = fabricated | Could be search error | Report as NEEDS_REVIEW |
| Rushing batch verification | Increases false positives/negatives | Limit batch size to 5 |
| Writing prose instead of table | Hard to scan, inconsistent | Use exact table format |
| Using own match quality terms | Inconsistent, confusing | Use EXACT/HIGH/MEDIUM/LOW/NONE/NEEDS_REVIEW |
| Skipping confidence percentage | Hides uncertainty level | Always provide 0-100% confidence |
| Not showing search queries | Can't reproduce verification | Show exact query strings used |

## Separating Citation vs. Claim Verification

**Citation verification:** Does "Smith et al., 2023, Nature" exist?
**Claim verification:** Does that paper actually say "95% efficacy"?

**This skill covers citation verification only.**

If user asks about claims:
1. First verify citation exists
2. Then note: "Claim verification requires reading the paper. I can download and extract it if you provide the DOI."
3. Use paper-ladder PDF extraction for claim verification

## Example Workflow

**Complete end-to-end example with paper-ladder:**

```python
import asyncio
from paper_ladder.clients import OpenAlexClient, CrossrefClient, SemanticScholarClient
from paper_ladder.extractors import PDFExtractor

async def verify_document_citations(pdf_path: str):
    """
    Complete workflow: Extract PDF → Parse citations → Verify → Report

    Usage:
        asyncio.run(verify_document_citations("paper.pdf"))
    """

    # Step 1: Extract text from PDF
    print("📄 Extracting PDF content...")
    extractor = PDFExtractor()
    content = await extractor.extract(pdf_path)
    text = content.markdown

    # Step 2: Extract references section
    print("🔍 Finding references section...")
    import re
    refs_match = re.search(r'(?i)^#+\s*References\s*$', text, re.MULTILINE)
    if refs_match:
        refs_text = text[refs_match.end():]
    else:
        refs_text = text  # Use entire document if no section found

    # Step 3: Parse individual citations
    print("📋 Parsing citations...")
    citations = []
    for line in refs_text.split('\n'):
        line = line.strip()
        if len(line) < 20:
            continue

        # Remove numbering
        line = re.sub(r'^\[?\d+\]?\.?\s*', '', line)

        # Extract metadata
        citation = {'raw': line}

        # Extract year
        year_match = re.search(r'\((\d{4})\)', line)
        if year_match:
            citation['year'] = year_match.group(1)

        # Extract title (in quotes)
        title_match = re.search(r'"([^"]+)"', line)
        if title_match:
            citation['title'] = title_match.group(1)

        # Extract DOI
        doi_match = re.search(r'doi:?\s*(10\.\d+/[^\s]+)', line, re.IGNORECASE)
        if doi_match:
            citation['doi'] = doi_match.group(1)

        citations.append(citation)

    print(f"✅ Found {len(citations)} citations\n")

    # Step 4: Verify each citation
    print("🔬 Verifying citations against databases...\n")

    async with OpenAlexClient() as oa, \
               CrossrefClient() as cr, \
               SemanticScholarClient() as s2:

        for i, cite in enumerate(citations, 1):
            print(f"## Citation {i}: {cite.get('title', 'Unknown')[:50]}...")

            # Build search query
            if 'doi' in cite:
                query = cite['doi']
            elif 'title' in cite:
                query = f'"{cite["title"]}"'
            else:
                query = cite['raw'][:100]

            # Search all databases
            oa_results = await oa.search(query, limit=3)
            cr_results = await cr.search(query, limit=3)
            s2_results = await s2.search(query, limit=3)

            # Determine match quality
            if oa_results or cr_results or s2_results:
                print("✓ Found in databases")
                # Compare metadata here...
            else:
                print("✗ NOT FOUND - Likely fabricated")

            print()

    return citations

# Run the verification
asyncio.run(verify_document_citations("paper.pdf"))
```

**Quick verification of a single citation:**

```python
async def quick_verify(title: str, year: str = None):
    """Quick single citation verification."""
    async with OpenAlexClient() as oa:
        results = await oa.search(f'"{title}"', limit=1)

        if results:
            paper = results[0]
            print(f"✓ Found: {paper.title}")
            print(f"  Authors: {', '.join(a.name for a in paper.authors[:3])}")
            print(f"  Year: {paper.year}")
            print(f"  DOI: {paper.doi}")

            if year and str(paper.year) != str(year):
                print(f"  ⚠️ Year mismatch: expected {year}, found {paper.year}")
        else:
            print("✗ Not found")

# Usage
asyncio.run(quick_verify("Attention Is All You Need", "2017"))
```

## Real-World Impact

**False negatives (real papers marked "not found"):**
- User removes valid citations
- Weakens manuscript/grant
- Wastes time re-searching

**False positives (fake papers marked "verified"):**
- User keeps fabricated citations
- Research integrity violation
- Manuscript rejection
- Career damage

**Both are worse than "needs human review".**

When uncertain, always recommend human review with evidence for user to investigate.

---
> Source: [SteadfastAsArt/citation-verification-skill](https://github.com/SteadfastAsArt/citation-verification-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
