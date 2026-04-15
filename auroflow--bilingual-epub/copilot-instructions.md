## bilingual-epub

> A command-line Python tool that takes two EPUB files (an original and its translation) and produces a single combined EPUB where each paragraph from the original is immediately followed by its translated counterpart. The output preserves the original book's formatting, structure, and pagination.

# CLAUDE.md — Bilingual EPUB Creator

## Project Overview

A command-line Python tool that takes two EPUB files (an original and its translation) and produces a single combined EPUB where each paragraph from the original is immediately followed by its translated counterpart. The output preserves the original book's formatting, structure, and pagination.

## Quick Reference

- **Language:** Python 3.10+
- **Package manager:** pip with pyproject.toml
- **Entry point:** `python -m bilingual_epub <original.epub> <translated.epub> -o output.epub`
- **Test command:** `pytest tests/`
- **Lint:** `ruff check src/`
- **Format:** `ruff format src/`

## Project Structure

```
bilingual-epub/
├── CLAUDE.md
├── pyproject.toml
├── README.md
├── src/
│   └── bilingual_epub/
│       ├── __init__.py
│       ├── cli.py                # Argument parsing, main entry point
│       ├── epub_parser.py        # EPUB unpacking, spine reading, paragraph extraction
│       ├── alignment.py          # Paragraph alignment engine (pluggable strategy)
│       ├── similarity.py         # Similarity scoring functions
│       ├── merger.py             # Combines aligned pairs into new XHTML documents
│       └── epub_writer.py        # Assembles and writes the final EPUB file
└── tests/
    ├── conftest.py               # Shared fixtures (sample EPUB builders)
    ├── test_epub_parser.py
    ├── test_alignment.py
    ├── test_similarity.py
    ├── test_merger.py
    ├── test_epub_writer.py
    └── test_integration.py       # End-to-end: two EPUBs in, one bilingual EPUB out
```

## Dependencies

- `ebooklib` — reading and writing EPUB files
- `beautifulsoup4` with `lxml` — parsing and manipulating XHTML content
- `numpy` — matrix operations for alignment dynamic programming
- `click` — CLI framework
- `pytest` — testing
- `ruff` — linting and formatting

Do not add any dependency on LLM APIs or large ML models. Alignment must work offline with classical algorithms.

## Architecture and Data Flow

### Pipeline

```
Original EPUB ─► parse ─► flat list of ParagraphNodes ─┐
                                                        ├─► align ─► merged XHTML ─► output EPUB
Translated EPUB ─► parse ─► flat list of ParagraphNodes ─┘
```

### Step 1: EPUB Parsing (`epub_parser.py`)

An EPUB is a ZIP archive. Use `ebooklib` to open it and read the OPF spine, which defines the reading order of XHTML documents. For each XHTML document in spine order, use BeautifulSoup to walk the DOM and extract block-level text elements.

Define a `ParagraphNode` dataclass:

```python
@dataclass
class ParagraphNode:
    text: str                  # Plain text content (for alignment)
    html_element: Tag          # Original BeautifulSoup Tag (preserves formatting)
    source_file: str           # Which XHTML file this came from
    index_in_file: int         # Position within that file
    global_index: int          # Position across the entire book
    element_tag: str           # e.g. "p", "h1", "blockquote"
```

Extract from these tags: `p`, `h1`–`h6`, `blockquote`, `li`, and `div` elements that directly contain text (not wrapper divs). Skip elements that contain no meaningful text (e.g., purely whitespace, or elements containing only images). Preserve image-only elements in a separate list so they can be reinserted at the correct positions in the output.

**Critical for Challenge #2:** Flatten all paragraphs from all XHTML files in spine order into a single list per book. This erases the arbitrary file-boundary differences between the two editions. Pagination boundaries are reintroduced later based on the original book's file structure.

### Step 2: Paragraph Alignment (`alignment.py`, `similarity.py`)

This is the core algorithmic challenge. The two books may have slightly different paragraph counts due to translation choices, editorial changes, or redactions. We need to find the best global alignment between the two paragraph sequences.

#### Similarity Scoring (`similarity.py`)

Implement a pluggable `SimilarityScorer` protocol:

```python
class SimilarityScorer(Protocol):
    def score(self, a: ParagraphNode, b: ParagraphNode) -> float:
        """Return similarity score in [0, 1]. Higher = more similar."""
        ...
```

**Primary implementation: `HybridSimilarityScorer`**

This combines multiple lightweight signals. Each signal produces a score in [0, 1], and the final score is a weighted sum.

1. **Length ratio (weight: 0.4):** The character counts of translated paragraph pairs are roughly proportional. Compute `score = 1 - abs(len_a * expected_ratio - len_b) / max(len_a * expected_ratio, len_b)` where `expected_ratio` is estimated from the median length ratio of all paragraphs in both books (compute this once upfront as a preprocessing step). This is inspired by the Gale-Church algorithm, which is the gold standard for bilingual sentence alignment in computational linguistics.

2. **Shared tokens (weight: 0.3):** Numbers, proper nouns, dates, URLs, and punctuation patterns often survive translation unchanged. Extract all tokens matching `\d+[\d.,]*`, capitalized words, and special punctuation sequences from both paragraphs. Compute Jaccard similarity over these token sets. This is especially powerful because names like "Anna Karenina" or numbers like "1865" appear identically in both languages.

3. **Structural similarity (weight: 0.2):** Compare the HTML tag type (h1 matches h1, p matches p). Heading-to-heading alignment should be strongly preferred. Also compare approximate position within the overall book (normalized global_index). Score = 1 if same tag and within 5% positional distance, decaying otherwise.

4. **Paragraph fingerprint (weight: 0.1):** Count sentences (by splitting on `.!?`), count sub-elements (bold, italic, links), compare these counts. Translators usually preserve paragraph-internal structure.

**Alternative implementations to support in the future (document these as TODOs):**
- `EmbeddingSimilarityScorer` using multilingual sentence-transformers (e.g., `paraphrase-multilingual-MiniLM-L12-v2`) for true semantic similarity — higher quality but requires a model download.
- `DictionaryScorer` using a bilingual dictionary for word-overlap scoring.

#### Alignment Algorithm (`alignment.py`)

Use a **modified Needleman-Wunsch algorithm** (global sequence alignment via dynamic programming), adapted for paragraph alignment. This is the same family of algorithms used in bioinformatics for DNA sequence alignment and in NLP for bilingual text alignment.

Define the scoring:
- **Match:** `+similarity_score(a, b)` (from the scorer above)
- **Gap in original (paragraph exists only in translation):** penalty of `-0.3`
- **Gap in translation (paragraph exists only in original):** penalty of `-0.3`
- **Merge (two paragraphs in one language align to one in the other):** Allow 2-to-1 and 1-to-2 merges by considering, at each cell, not just `(i-1, j-1)`, `(i-1, j)`, `(i, j-1)` but also `(i-2, j-1)` and `(i-1, j-2)`. Score the merge by computing similarity of the concatenated paragraphs against the single paragraph.

The DP matrix is `(N+1) x (M+1)` where N and M are the paragraph counts. For very long books (10,000+ paragraphs), this could be expensive. Optimize with:
- **Banded alignment:** Since paragraphs are roughly in order, restrict the DP to a diagonal band (e.g., bandwidth = max(100, 0.1 * max(N, M))). This reduces complexity from O(NM) to O(N * bandwidth).
- **Anchor-based chunking:** First, find high-confidence anchor points by matching chapter headings (h1/h2 elements) or paragraphs with very high similarity. Then run alignment independently within each chunk between anchors. This makes the problem embarrassingly parallelizable and much faster.

The output of alignment is a list of `AlignedPair`:

```python
@dataclass
class AlignedPair:
    original: list[ParagraphNode]    # 0 or more (0 = gap, 2+ = merge)
    translated: list[ParagraphNode]  # 0 or more
    confidence: float                # alignment confidence score
```

### Step 3: Merging (`merger.py`)

Take the list of `AlignedPair` objects and produce new XHTML documents.

For each aligned pair:
1. Emit the original paragraph's HTML element(s) with a CSS class `bilingual-original`.
2. Immediately after, emit the translated paragraph's HTML element(s) with a CSS class `bilingual-translated`.
3. If one side is a gap (no matching paragraph), emit only the side that exists, with a class `bilingual-unmatched`.
4. Optionally insert a thin `<hr class="bilingual-separator"/>` between pairs for visual clarity.

**Re-pagination (Challenge #2):** Group the aligned pairs back into XHTML files following the original book's file boundaries. Use `ParagraphNode.source_file` from the original paragraphs: whenever the source file changes, start a new XHTML document in the output. This preserves the original's chapter structure, TOC references, and internal links.

**Preserving non-text content (Challenge #3):** Between paragraphs, the original XHTML may contain images, SVGs, tables, or other non-text block elements. During extraction, record these elements and their positions. During merging, reinsert them at the correct locations. Similarly, inline elements (bold, italic, links, ruby annotations) within paragraphs are preserved because we carry the full `html_element` Tag through the pipeline.

### Step 4: EPUB Assembly (`epub_writer.py`)

Construct the output EPUB by copying the original EPUB's structure and replacing the XHTML content files:

1. Copy all metadata from the original (OPF, NCX/nav, cover image, fonts).
2. Copy all CSS files from the original. Append a bilingual stylesheet with rules for `.bilingual-translated` (e.g., `color: #555; font-style: italic; margin-left: 1em; border-left: 2px solid #ccc; padding-left: 0.5em;`) and `.bilingual-separator`. The translated text styling should be visually distinct but not distracting.
3. Copy all images and other media from the original.
4. Replace the XHTML content files with the merged versions.
5. Preserve the spine order, TOC, and all internal cross-references from the original.

Use `ebooklib` to write the final EPUB. Validate it opens correctly in common readers (Calibre, Apple Books).

## CLI Interface

```
Usage: python -m bilingual_epub [OPTIONS] ORIGINAL TRANSLATED

Arguments:
  ORIGINAL    Path to the original EPUB file
  TRANSLATED  Path to the translated EPUB file

Options:
  -o, --output PATH          Output EPUB path [default: bilingual_output.epub]
  --original-lang TEXT       Language code of original (e.g., "en") [auto-detected if omitted]
  --translated-lang TEXT     Language code of translation (e.g., "ja") [auto-detected if omitted]
  --strategy TEXT            Alignment strategy: "hybrid" [default: "hybrid"]
  --band-width INT           Alignment band width for DP [default: auto]
  --translated-style TEXT    CSS for translated paragraphs [default: built-in]
  --debug                    Write alignment debug info to stderr
  --dump-alignment PATH      Write alignment pairs as JSON for inspection
  -v, --verbose              Verbose logging
```

## Testing Strategy

- **Unit tests** for each module in isolation. Use small, hand-crafted XHTML snippets and paragraph lists.
- **`test_epub_parser.py`:** Build minimal EPUBs programmatically using `ebooklib`, verify correct paragraph extraction and ordering across multiple XHTML files.
- **`test_similarity.py`:** Test each similarity signal independently with known paragraph pairs. Test that heading-to-heading scores high, heading-to-body scores low, matching numbers boost score, etc.
- **`test_alignment.py`:** Test Needleman-Wunsch with small synthetic sequences. Include cases with perfect 1:1, a gap on one side, a 2:1 merge, and a completely missing chapter. Verify the banded optimization produces identical results to unbanded for cases within the band.
- **`test_merger.py`:** Verify that merged XHTML contains interleaved paragraphs with correct CSS classes, that non-text elements are preserved, and that file boundaries follow the original.
- **`test_epub_writer.py`:** Verify the output is a valid EPUB (ZIP structure, required files present, valid XHTML).
- **`test_integration.py`:** Full pipeline test with a small two-chapter book where the translation has one extra paragraph in chapter 1 and one missing paragraph in chapter 2. Verify the output EPUB can be opened and has the expected structure.

Build a `conftest.py` fixture that can programmatically create minimal EPUB files with specified content for testing, so tests don't depend on external book files.

## Key Design Principles

- **Pluggability of alignment strategy.** The scorer and aligner are behind protocols/abstract base classes so new strategies (embedding-based, dictionary-based) can be swapped in without touching the rest of the pipeline.
- **Separation of text extraction from HTML structure.** Plain text is used for alignment scoring, but the original HTML Tags are carried through so formatting is never lost.
- **Flatten then re-paginate.** This cleanly separates the alignment problem (which operates on flat sequences) from the structural problem (which only matters for output).
- **No LLM dependency.** Everything runs locally and offline. The hybrid length + shared-tokens + structure approach is well-established in computational linguistics and works reliably for paragraph-level alignment.

## Implementation Order

1. `epub_parser.py` — get paragraphs out of EPUBs (and write tests)
2. `similarity.py` — implement `HybridSimilarityScorer` (and write tests)
3. `alignment.py` — implement Needleman-Wunsch with banding and anchoring (and write tests)
4. `merger.py` — interleave aligned paragraphs into XHTML (and write tests)
5. `epub_writer.py` — assemble the output EPUB (and write tests)
6. `cli.py` — wire everything together
7. `test_integration.py` — end-to-end test
8. Polish: error handling, edge cases, logging, progress reporting

## Edge Cases to Handle

- Books where one side has a preface/foreword that the other doesn't — the aligner should gap through it.
- Paragraphs that are split differently (one long paragraph in original becomes two in translation) — 1:2 merge handling.
- Chapters in different order — the anchor-based approach handles this if chapter headings can be matched; otherwise the tool should warn the user.
- Non-text-heavy pages (e.g., title page, copyright page, table of contents) — pass these through from the original without attempting alignment.
- Very short paragraphs (single words, numbers) — these are hard to align in isolation; rely on surrounding context via the DP algorithm.
- EPUB2 vs EPUB3 differences — `ebooklib` abstracts most of this but test with both.
- RTL languages — CSS and HTML `dir` attributes must be preserved.
- Ruby annotations (common in Japanese) — these are inline elements and should be preserved inside paragraph Tags.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/auroflow) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
