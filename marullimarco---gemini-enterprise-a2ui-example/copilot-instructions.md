## gemini-enterprise-a2ui-example

> **Persona:** Data Science Partner & Demo Architect

# Project: Generic Pricing Competitor Agent Dashboard

**Persona:** Data Science Partner & Demo Architect
**Context:** Preparing a high-stakes demo for the Retail Pricing Director this Wednesday.
**Objective:** Architect a synthetic data environment and functional agents to showcase the Gemini Enterprise (GE) + BigQuery (BQ) integration paths.

---

## Layer 1: Directive (The Protocol)

### 1. The Living Demo Mission
This document is a **living protocol**. As we iterate with the agents to finalize the most impactful use cases, we will return here to update the schemas and success metrics. The current focus is strictly on the Pricing Competitor Agent:

*   **Use Case 1: The Pricing Competitor Agent (Custom Agents via MCP)**
    *   *Path:* Using the BigQuery MCP Server.
    *   *Goal:* An agent that dynamically flags products that are priced higher than competitors (by fetching BQ pricing) and drafts pricing adjustment recommendations using synthetic data.
    *   *Current Status (Recovered & Operational):* Deployed as `generic-pricing-competitor-agent` utilizing standard A2A JSON-RPC integration (`/a2a/v1/json-rpc`). It processes file uploads directly to GCS (`<YOUR_UPLOAD_BUCKET_NAME>`) using customized V4 POST policies to avoid browser multipart rejections. Integrates cleanly with Vertex AI (`gemini-2.5-flash`) via the `GOOGLE_GENAI_USE_VERTEXAI` flag and is successfully registered to the regional `us` Gemini Enterprise App.`

### 2. Standard Operating Procedures (SOPs)
1.  **Research & Scrape:** Pull real product categories/brands to ensure the LLM recognizes native retail terms.
2.  **Iterative Refinement:** We will test these use cases against the "Director's Persona." If a use case doesn't land, we update this `Gemini.md` and re-generate the underlying data.
3.  **Provision & Meta-Tag:** Build BQ tables with verbose column descriptions. Agents rely on metadata to "understand" what a column like `last_purchase_date` means for churn.

---

## Layer 2: Orchestration (The Partner)

### 1. Evaluation & The "Improvement Loop"
*   **The Iteration Mandate:** We do not settle on the first draft of the data. We will query the agents, evaluate the quality of the insights, and **improve the data structures** until the "Demo Story" is seamless.
*   **Strategic Routing:** I will route the "Product Catalog" task to a scraping script and the "Behavioral Logic" (Churn/Sales) to a probabilistic generator.

### 2. Success Metrics
*   **Agent Accuracy:** The agent should correctly interpret a prompt like *"Show me the Northeast stores where cat food sales are lagging"* without needing specific SQL table names.
*   **UI Versatility:** Insights must be "executive-ready" on both **Gemini Web** and **Mobile** layouts.

---

## Layer 3: Execution (The Compute)

### 1. Scripts & Assets (`/projects/pricing-demo/scripts/`)
*   `research_products.py`: Scrapes product taxonomies to ensure realistic SKUs.
*   `synthetic_gen.py`: Generates the local datasets for the Pricing Agent Use Case.
*   `bq_provisioning.py`: Provisions the BQ dataset and handles the "Authoring in BQ" table limits (max 10 tables for quality control).

### 2. Validation Notebooks (`/projects/pricing-demo/notebooks/`)
*   `use_case_iteration.ipynb`: A workspace to "interrogate" the synthetic data with natural language before the final BQ upload.

---

## Layer 3.5: Dynamic Presentation (A2UI Delivery)

### 1. The A2UI Communication Protocol
The agent delivers interactive UI elements (like charts or forms) by appending standard A2UI primitives to its plain-text responses.

*   **Format Constraints:**
    *   Interactive payloads are JSON arrays wrapped in `<a2ui-json>` and `</a2ui-json>` tags.
    *   Uses MIME Type `application/json+a2ui`.
*   **Parsing Pipeline (`executor.py`):**
    *   The backend splits the LLM output into standard `TextPart` and `DataPart` (A2UI) using regex. This matches the official Google ADK A2A standards.
*   **A2UI Primitives Used:**
    *   `VegaChart`: For rich analytics (e.g. Nutella county-level pricing comparisons).
    *   `WebFrameSrcdoc`: For custom iframe-based widgets, such as file upload forms directly to GCS.

---

## Layer 4: Platform UI Constraints & Security

### 1. Same-Origin Policy for `WebFrameSrcdoc`
Empirical testing confirms that modern browser sandboxing prevents the `WebFrameSrcdoc` (which is served from isolated `usercontent.goog` domains) from accessing or modifying the outer platform's DOM (`window.parent.document`). 

Any attempts to traverse up and modify the `ucs-search-bar` or `file-picker-button` are **blocked by the browser's Same-Origin Policy**.
*   **Conclusion:** Guest agents cannot alter the application chrome or hide outer platform buttons. This feature toggle must be supported natively by the Gemini Enterprise platform settings.

---

## Workspace Organization

```text
/projects/pricing-bq-demo/
├── data/               # Local CSVs and scraped JSON
├── notebooks/          # Dry-run query validation & Use Case iteration
├── scripts/            # Scrapers, Synthesizers, and BQ Loaders
└── reports/            # The "Wednesday Demo Script" and Storyboard
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marullimarco) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
