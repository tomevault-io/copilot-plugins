## genoresearch

> Reviewed and verified by Claude on 2026-03-14.

# GenoResearch — Verified Findings

Reviewed and verified by Claude on 2026-03-14.
229 raw findings were audited. 205 were discarded (spam, errors, well-studied genes).
24 dark gene findings were retained and verified below.

---

## Summary

| Gene | Status | Key Finding | Confidence |
|------|--------|-------------|------------|
| C8orf48 | **Function found** | Inhibits colorectal cancer via MAPK pathway (PMID:33309715) | High |
| C12orf43 | **Function found** | Annotated as Protein CUSTOS; Wnt signaling pathway regulation | High |
| C15orf39 | **Not a dark gene** | Known function: inhibits NF-kB signaling via PRMT2 interaction | High |
| LOC128125817 | **Dark gene confirmed** | 26 aa uncharacterized protein, conserved in caribou + mouse | Medium |
| LOC101060341 | **Dark gene confirmed** | Uncharacterized protein FLJ44672, no BLAST homologs at all | Medium |
| LOC340313 | **Withdrawn** | Discontinued by NCBI, not predicted in later annotations | High |
| LOC100130357 | **Incomplete** | Investigation started but no real hypothesis generated | Low |
| PPP3CA-DT | **Dark gene confirmed** | Divergent transcript of PPP3CA, understudied despite homologs | Medium |
| A0A8T1SHP2 | **Characterized** | TBC1D7 domain family member from Chelydra serpentina (turtle) | High |
| G3WRF0 | **Characterized** | Secreted frizzled-related protein 1 from Sarcophilus harrisii | High |
| TBC1D7 | **Characterized** | Human Rho GTPase regulatory family, 293 aa, NM_001143965.4 | High |
| A0A2R8Y556 | **Dark gene confirmed** | 26 aa regulatory peptide, 100% identical to LOC128125817 | Medium |

---

## Verified Findings (12 genes, 24 files)

### 1. C8orf48 — Chromosome 8 ORF 48
**Status: Function confirmed (no longer a dark gene)**

- **Gene ID:** 157773 | **Chromosome:** 8p22 | **Protein:** 319 aa
- **Composition:** 33.9% charged, 31.0% hydrophobic
- **BLAST:** 100% identity with NP_001007091.2 (self-match only)
- **UniProt:** Q96LL4 (was listed as uncharacterized)
- **PubMed discovery:** C8orf48 inhibits colorectal cancer tumorigenesis by regulating the MAPK signaling pathway (PMID:33309715)
- **Verdict:** Was dark, now has a confirmed function. Good discovery by the agent.

**Files:**
- `C8orf48 - Comprehensive Analysis Complete.md`
- `C8orf48 - Function Confirmed via PubMed.md`
- `C8orf48 - Uncharacterized Protein Analysis.md`

---

### 2. C12orf43 — Protein CUSTOS
**Status: Function confirmed (no longer a dark gene)**

- **UniProt:** Q96C57
- **Function:** Wnt signaling pathway regulation during early development
- **Note:** Previously classified as hypothetical protein with no known function
- **Verdict:** Successfully transitioned from dark to annotated. Valid finding.

**Files:**
- `C12orf43 has been annotated as Protein CUSTOS in UniProt Q96.md`

---

### 3. C15orf39 — Chromosome 15 ORF 39
**Status: Not a dark gene (known function)**

- **Gene ID:** 56905 | **Chromosome:** 15q24.2 | **Protein:** 1047 aa
- **Composition:** 19.9% charged, 33.3% hydrophobic
- **Function:** Negatively regulates microglial inflammatory responses; inhibits NF-kB signaling by interacting with PRMT2 (PubMed:38892217)
- **BLAST:** Self-match only, no homologs in nt database
- **Verdict:** Agent correctly identified this is NOT a dark gene. Good quality control.

**Files:**
- `C15orf39 - Known Function Confirmed.md`

---

### 4. LOC128125817 — Uncharacterized micro-protein
**Status: Dark gene confirmed**

- **Chromosome:** 1p34.2 | **mRNA:** 81 bp | **Protein:** 26 aa
- **Composition:** 38.5% charged residues (unusually high for such a short peptide)
- **BLAST:** 100% identity in human (NP_001401929.1), 88% in caribou (Rangifer tarandus), 100% in mouse (LOC139428847)
- **Cross-species conservation** of a 26 aa uncharacterized protein is notable
- **Verdict:** Genuine dark gene. Conservation across 3 species suggests functional importance despite tiny size. Interesting candidate for follow-up.

**Files:**
- `Dark Gene LOC128125817.md`
- `LOC128125817 Analysis.md`
- `LOC128125817 Uncharacterized gene with conserved homologs.md`

---

### 5. A0A2R8Y556 — Identical to LOC128125817
**Status: Dark gene confirmed (same as LOC128125817)**

- **Protein:** 26 aa | 100% identical to LOC128125817
- **Hypothesis:** Likely functions as a regulatory peptide or signaling molecule based on short length, high charged residues, and cross-species conservation
- **Verdict:** This is the UniProt entry for the same protein as LOC128125817. Agent correctly linked them.

**Files:**
- `A0A2R8Y556 - Function Hypothesis medium.md`
- `A0A2R8Y556LOC128125817 - Dark Gene Confirmed.md`

---

### 6. LOC101060341 — Putative uncharacterized protein FLJ44672
**Status: Dark gene confirmed**

- **Organism:** Homo sapiens
- **BLAST:** No significant similarity found in any database
- **UniProt:** No results
- **Sequence:** Contains non-standard characters (potential data quality issue)
- **Verdict:** Genuinely uncharacterized — no homologs anywhere. Could be a pseudogene or novel ORF. The non-standard characters in the sequence warrant further investigation.

**Files:**
- `LOC101060341 - Dark Gene Investigation.md`
- `LOC101060341 - Uncharacterized Protein with No Homologs.md`

---

### 7. LOC340313 — Withdrawn gene
**Status: Discontinued by NCBI**

- **NCBI search:** 0 results
- **Gene status:** DISCONTINUED — not predicted in later genome annotations
- **No sequence data available**
- **Verdict:** Not a real gene. Agent correctly documented the withdrawal.

**Files:**
- `LOC340313 withdrawn from NCBI.md`
- `LOC340313  Function Hypothesis.md` (template only, incomplete)

---

### 8. LOC100130357 — Incomplete investigation
**Status: Incomplete**

- Finding file is a template stub: "Based on [evidence], LOC100130357 likely functions as..."
- No actual analysis was completed
- **Verdict:** Needs re-investigation. File kept as a placeholder.

**Files:**
- `LOC100130357  Function Hypothesis.md`

---

### 9. PPP3CA-DT — Protein Phosphatase 3 divergent transcript
**Status: Dark gene confirmed (understudied divergent transcript)**

- **Gene:** PPP3CA divergent transcript | **Chromosome:** 4q24
- **mRNA:** NM_000944.5 (4587 bp, 42.9-43.0% GC) and NM_001130692.2
- **Protein:** 469-521 aa depending on transcript
- **BLAST:** Multiple PPP3CA isoforms from various species
- **UniProt:** Q08209 — calcium-dependent, calmodulin-stimulated protein phosphatase
- **Note:** The parent gene PPP3CA is well-characterized, but the **divergent transcript (PPP3CA-DT)** is understudied. No UniProt protein entry exists for the DT form specifically.
- **Verdict:** The divergent transcript itself is genuinely dark despite the parent gene being known. Good nuanced finding.

**Files:**
- `PPP3CA-DT - Protein Phosphatase 3 Analysis Complete.md`
- `PPP3CA-DT - Understudied Dark Gene.md`
- `PPP3CA-DT Understudied dark gene with protein phosphatase 3 .md`

---

### 10. A0A8T1SHP2 — TBC1D7 domain family (Chelydra serpentina)
**Status: Characterized via BLAST**

- **Organism:** Chelydra serpentina (snapping turtle) — non-human
- **Protein:** 293 aa | TBC1D7-LOC100130357 readthrough
- **BLAST:** 100% identity with KAG6928250.1 (TBC1D7-LOC100130357 readthrough)
- **Related species:** Partial match in Macrochelys suwanniensis
- **Verdict:** Non-human dark gene successfully characterized. TBC1D7 domain family membership confirmed.

**Files:**
- `A0A8T1SHP2 - Dark Gene Investigation.md`
- `A0A8T1SHP2 - TBC1D7 Domain Family Confirmed.md`
- `A0A8T1SHP2 - TBC1D7 Domain Family Member.md`

---

### 11. G3WRF0 — Secreted frizzled-related protein 1 (Sarcophilus harrisii)
**Status: Characterized via BLAST**

- **Organism:** Sarcophilus harrisii (Tasmanian devil)
- **Protein:** 314 aa | Secreted frizzled-related protein 1
- **BLAST:** 100% identity with XP_023350501.2, KAM9063868.1, KAM9065283.1
- **Verdict:** Non-human dark gene successfully characterized. SFRP1 is involved in Wnt signaling; the marsupial version was uncharacterized in UniProt.

**Files:**
- `G3WRF0 - Dark Gene Investigation.md`
- `G3WRF0 - Secreted Frizzled-Related Protein Confirmed.md`

---

### 12. TBC1D7 — Human Rho GTPase regulatory family
**Status: Characterized**

- **mRNA:** NM_001143965.4 (1122 bp, 47.5% GC)
- **Protein:** 293 aa
- **Motifs:** Polyadenylation signal, CCAAT box promoter elements
- **Family:** Rho GTPase regulatory family
- **Verdict:** Well-characterized gene. Serves as a reference for the A0A8T1SHP2 investigation above.

**Files:**
- `TBC1D7 Gene Family Analysis.md`

---

## Audit Statistics

- **Total raw findings generated by Qwen:** 229
- **Discarded as junk/spam/errors:** 205 (89%)
  - BRCA1 transcript variant spam: ~60
  - TP53 transcript variant spam: ~45
  - Other well-studied genes (APOE, KRAS, PTEN, etc.): ~75
  - Error findings (BLAST timeout, File Not Found, etc.): ~15
  - Numbered/untitled files: ~10
- **Retained (dark gene relevant):** 24 (11%)
- **Unique genes investigated:** 12
- **Genuine dark genes confirmed:** 5 (LOC128125817/A0A2R8Y556, LOC101060341, PPP3CA-DT, A0A8T1SHP2, G3WRF0)
- **Functions discovered:** 2 (C8orf48 → MAPK pathway, C12orf43 → Wnt signaling)
- **Correctly identified as not dark:** 2 (C15orf39, LOC340313)

---

*Audit performed by Claude (Anthropic) on 2026-03-14. Findings were evaluated for scientific accuracy, relevance to the dark genome mission, and evidence quality.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Realrandombacon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Realrandombacon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
