## mining-articles-extract

> <edge_cases_and_learnings>


# Rule: M&A Data Extraction - Supplement 1 (Edge Cases & Learnings)
# Activation: Always On
# Description: Contains specific, learned edge cases and data hygiene rules that supplement the core extraction logic.

<edge_cases_and_learnings>
- **Scan Appendices & JORC Tables:** Crucial data, especially `coverage_area_raw`, is often hidden in appendices like "JORC Code - Table 1". You must scan the 'Commentary' column of these tables.

- **Data Relevance:** All data MUST match the current project. You must ignore data that is explicitly for other projects, even if it's in the same document.
  - **NEW JUSTIFICATION REQUIREMENT:** When a potential value is ignored due to this rule, the `justification` for the `null` field MUST explicitly state that the data was for the wrong project and MUST name the project(s) the data actually belongs to. This applies to every field where a candidate value was considered and rejected.

- **Distinguish Current vs. Historical/Legacy Terms:** Focus your extraction on the "Consideration" or "Transaction Terms" of the NEW deal being announced. You must ignore financial details described as "legacy," "historical," or from a prior agreement that is now being completed or superseded.

- **Cash-for-Equity vs. Share Swaps:** If Company A pays cash to Company B to receive shares *in Company B*, this is a `cash_payments_raw` from Company A's perspective. If Company A pays by giving shares *of its own stock (Company A stock)*, it is a `share_payments_raw`.

- **Aggregate All Payment Components (CRITICAL):** You must sum all forms of a payment type into a single total. This includes future, conditional, or milestone-based payments.
  - **Example:** If a deal is for "$250,000 cash and a milestone payment of $1,000,000", the `cash_payments_raw` value MUST be `1250000`.
  - **Example:** If a deal is for "$250k in shares and an additional $200k in shares upon a milestone", the `share_payments_raw` value MUST be `450000`.

- **NSR in Transaction:** The `nsr_acquired_percent` field should be populated if an NSR is part of the transaction's terms. This includes both NSRs *acquired* by the buyer and NSRs *retained* by the seller. The justification must clarify whether the NSR was acquired or retained.

- **Link/Content Mismatch:** If a source link is for Project A but the text content is about Project B, you must report this as a critical error and stop.

- **Shared Commitments:** If a value is shared by N projects, the `value` you extract should be `RAW_VALUE / N`.

- **Farm-in Agreements:** For `interest_acquired_percent`, you must use the final potential ownership percentage.

- **Data Hygiene:** If a field's value is not mentioned in the text, its `value` must be `null`. When extracting numbers, you must remove formatting like commas or currency symbols (e.g., "$1,500,000" becomes `1500000`).
</edge_cases_and_learnings>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mbessalle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
