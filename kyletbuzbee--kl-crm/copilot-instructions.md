## kl-crm

> This document provides a comprehensive overview of the K&L Recycling CRM project, designed to be used as instructional context for Gemini.

# K&L Recycling CRM - Project Context

This document provides a comprehensive overview of the K&L Recycling CRM project, designed to be used as instructional context for Gemini.

## 1. Project Overview

This is a production-grade, Google Apps Script-based CRM for K&L Recycling. Its primary purpose is to automate and manage prospect management, outreach tracking, and account conversion.

**Key Technologies:**
*   **Backend:** Google Apps Script (V8 Runtime)
*   **Frontend:** HTML Service (within Google Apps Script)
*   **Database:** Google Sheets
*   **Tooling:** Node.js for local testing, `clasp` for command-line deployment.

**Architecture:**
The system uses a 4-tier Service-Oriented Architecture (SOA) to ensure modularity and separation of concerns:
*   **Tier 1: Core Engine:** Low-level utilities, configuration, and data access (`Config.js`, `SharedUtils.js`, `SchemaNormalizer.js`).
*   **Tier 2: Business Services:** Implements business rules for CRM entities (`ProspectFunctions.js`, `OutreachFunctions.js`, `AccountFunction.js`).
*   **Tier 3: Workflow & Automation:** Orchestrates processes and timed triggers (`WorkflowAutomationService.js`, `Sync.js`, `PipelineService.js`).
*   **Tier 4: User Interface:** Frontend for user interaction (`CRM_Suite.html`, `DashboardBackend.js`, `CRM_Styles.html`, `CRM_Scripts.html`).

## 2. Building and Running

### Deployment

This is a Google Apps Script project. Deployment is handled via `clasp`, the command-line interface for Apps Script.

*   **Push Code:** `clasp push`
    *   This command uploads the local project files to the linked Google Apps Script project.
*   **Open Project:** `clasp open`
    *   This opens the Google Apps Script project in the browser.

### Testing

The project includes a simple test runner using Node.js.

*   **Run Tests:** `npm test`
    *   This command executes the `run_tests.js` script.
*   **Test Commands:**
    *   `npm run test` - Run default tests.
    *   `npm run test:help` - Show available test commands.

## 3. Development Conventions

Adherence to the existing conventions is critical for maintaining the stability and integrity of the CRM.

### Core Principles:
*   **Config-First:** Always use the `CONFIG` object from `Config.js` for sheet names and column headers. **Do not use "magic strings" or hardcoded values.**
*   **Schema-Driven:** Use `SchemaNormalizer.js` and `system-schema.json` as the single source of truth for field names and variations.
*   **Batch Operations:** To avoid Google Apps Script execution timeouts, all spreadsheet I/O must be done in batches. **Do not call `getValue` or `setValue` inside loops.**
*   **Safe Data Handling:**
    *   Use `SharedUtils.getSafeSheetData()` for reading data to ensure consistency and automatic row index tracking (`_rowIndex`).
    *   Use `SharedUtils.formatDate()` for all date formatting to ensure timezone consistency.
*   **Standardized Returns:** All functions should return a standard object: `{ success: boolean, data: any, error: string }`.
*   **ID-Based Lookups:** Rely on primary keys (`Company ID`, `Outreach ID`) for identifying and modifying records. Avoid relying on row indexes alone for long-term storage, although `_rowIndex` is used for immediate updates.

### Schema and Data Model
*   The master schema is defined in `system-schema.json`.
*   Sheet names and column headers are defined in `Config.js`. These must be kept in sync with the physical Google Sheets and the schema file.
*   The `SchemaNormalizer.js` utility is used to create a canonical mapping between different naming conventions (e.g., `Company Name` vs. `companyName`).

### Critical Logic Flows
*   **Outreach Sync:** New entries in the 'Outreach' sheet trigger updates to the corresponding 'Prospects' record via `updateProspectAfterVisit`.
*   **Account Conversion:** An "Account Won" outcome triggers a workflow that migrates a prospect's data to the 'Accounts' sheet.
*   **Urgency & Scoring:** Prospects are automatically scored and categorized into "Urgency Bands" (Overdue, High, Medium, Low) based on their next step due dates.
*   **Daily Maintenance:** A time-based trigger runs daily (`checkStaleProspects`) to identify stale prospects and perform cleanup tasks.

## 4. Key Files and Folders

*   `Config.js`: Central configuration for sheet names, headers, and app settings.
*   `SharedUtils.js`: Core utility library for safe data access, ID generation, and date handling.
*   `SchemaNormalizer.js`: Component for resolving naming inconsistencies and providing canonical field access.
*   `CRM_Suite.html`: The main entry point for the CRM's user interface.
*   `DashboardBackend.js`: Logic specifically for the dashboard UI interactions.
*   `ProspectFunctions.js` / `OutreachFunctions.js`: Primary business logic for core entities.
*   `package.json`: Local development and testing dependencies.
*   `appsscript.json`: Google Apps Script manifest and permissions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kyletbuzbee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
