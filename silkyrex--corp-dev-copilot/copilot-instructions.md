## corp-dev-copilot

> Read investment term sheets and merger agreements through a corp-dev lens. Extract key business terms, flag redlines, surface pushback-worthy issues. Use when the user says "read this term sheet", "review this merger agreement", "/read-deal-doc", or points to a deal doc (PDF, .docx, text).


# Read Deal Doc

The user is in corp dev. Legal owns the doc; the user owns the business items. Corp-dev lens, not legal lens.

## Input

PDF, .docx (preserve track changes if present), or pasted text.

## Output

Markdown file. Default path: `./deals/YYYY-MM-DD-<deal-slug>.md`. Override if the user gives one.

## Checklist — always emit one row per item, even if NOT FOUND

**Term sheets:**
1. Pro-rata rights
2. Financial information rights
3. Marketing rights / logo + name usage consent
4. Purchase price / valuation mechanics (pre/post, option pool treatment)

**Merger agreements:**
1. Purchase price + adjustment mechanics
2. Earnout
3. Closing conditions

Any other material business term present: include. Legal-owned items (reps, indemnity, governing law): tag `[legal-owned, FYI]`, one line only, unless unusual.

## Output template

```markdown
# <Deal> — <term sheet | merger agreement> review
Date: YYYY-MM-DD | Source: <path or "pasted">

## 1. Key Terms

### <Term>
- Language: "<quoted>" (§<section>)
- Plain English: <one sentence>
- Status: standard | non-standard | missing | needs attention

## 2. Redline Flags
(Only if redlined. Omit section otherwise.)
- <What changed> (§<section>) — favors: us | counterparty | neutral. Take: <1-2 sentences>

## 3. Issues & Pushback
Prioritized.
1. <Issue> — why it matters. Suggested pushback: "<language>"
```

## Report back

After writing the file: path + 3 bullets (biggest issue, most non-standard term, anything missing from the checklist).

---
> Source: [silkyrex/corp-dev-copilot](https://github.com/silkyrex/corp-dev-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
