## cti-agent

> To evolve the AI-driven command-line utility into a comprehensive and interactive threat intelligence platform. The agent will not only rapidly assess the security posture of software applications but also persist and leverage historical SBOM data for deeper analysis and trend identification. By enabling a conversational, AI-driven query interface, the agent will transform from a static analysis tool into a dynamic security advisor, allowing users to intuitively explore vulnerability data and gain deeper insights on demand. For version 4.0, the agent will significantly enhance its contextual understanding and defensive measure recommendations through advanced RAG (Retrieval Augmented Generation) capabilities and improved handling of large SBOMs.

# Product Requirements Document: Cyber Threat Intelligence Agent

Version: 4.0

Date: July 22, 2025

Status: DRAFT

## 1.0 Product Vision & Overview

### 1.1 Vision Statement

To evolve the AI-driven command-line utility into a comprehensive and interactive threat intelligence platform. The agent will not only rapidly assess the security posture of software applications but also persist and leverage historical SBOM data for deeper analysis and trend identification. By enabling a conversational, AI-driven query interface, the agent will transform from a static analysis tool into a dynamic security advisor, allowing users to intuitively explore vulnerability data and gain deeper insights on demand. For version 4.0, the agent will significantly enhance its contextual understanding and defensive measure recommendations through advanced RAG (Retrieval Augmented Generation) capabilities and improved handling of large SBOMs.

### 1.2 Project Goals & Objectives

The primary goal of the Cyber Threat Intelligence Agent is to transform the process of vulnerability management from a reactive, manual, and often overwhelming task into a proactive, automated, and intelligent workflow. This will be achieved through the following key objectives:

- Automate Vulnerability Discovery: The agent will eliminate the laborious and error-prone manual process of cross-referencing software components listed in an SBOM against disparate vulnerability databases. It will programmatically parse dependencies and initiate a comprehensive security analysis.

- Enhance Intelligence with AI: The project will leverage the advanced reasoning capabilities of Large Language Models (LLMs) to perform nuanced analysis tasks. This includes dynamically generating Common Platform Enumeration (CPE) strings, mapping vulnerabilities to adversary tactics (TTPs), and generating concise, human-readable summaries of complex security findings.

- Provide Actionable Context: The agent will move beyond simple vulnerability scanning to provide rich, actionable context. By correlating vulnerabilities with frameworks like MITRE ATT&CK and Common Weakness Enumeration (CWE), it will explain how a vulnerability might be exploited and what underlying software weakness it represents. Furthermore, it will furnish concrete defensive measures, such as detection rules in Sigma, Snort, or Yara formats, enabling immediate action.

- Implement Risk-Based Prioritization: A core objective is to generate a remediation plan that reflects true business risk. The agent will implement a sophisticated prioritization algorithm that incorporates a holistic view of risk, including static severity scores (CVSS), real-world exploits from the CISA KEV catalog, the predictive probability of exploitation from EPSS, and the potential impact derived from adversary tactics.

- Enable Interactive Analysis: The agent will introduce a "chatbot" feature, allowing users to conversationally query the stored SBOM and vulnerability data. This transforms the agent into an interactive tool for security exploration and makes threat intelligence more accessible.

- Developer-Centric Workflow: The entire experience is designed for the modern developer and security professional. By providing a seamless, powerful command-line interface (CLI), the agent integrates smoothly into existing development, security, and CI/CD workflows. Reports will be saved as structured Markdown files for easy sharing, archiving, and integration with other systems.

- Facilitate Historical Analysis: Lay the groundwork for analyzing changes in SBOMs over time and identifying evolving vulnerability trends.

- Simplify Local Deployment: Provide a straightforward method for local database deployment using Docker.

- **Improve LLM Context with RAG**: Implement Retrieval Augmented Generation (RAG) to provide LLMs with more relevant and up-to-date information, especially for defensive measures and CVE details.

- **Handle Large SBOMs**: Ensure the agent can process large SBOMs without encountering token limit issues with LLMs.

### 1.3 Target Audience

The Cyber Threat Intelligence Agent is designed for technical users who operate at the intersection of software development and cybersecurity.

- Primary Audience:

  - DevSecOps Engineers: Professionals responsible for embedding security practices and automated checks into the CI/CD pipeline. They require fast, reliable, and scriptable tools to assess application security without slowing down development cycles. They also need to track the security posture of applications over time, easily re-run analysis on previously submitted SBOMs without re-uploading, and provide historical context to development teams regarding dependency changes and vulnerability remediation efforts.

  - Application Security (AppSec) Specialists: Analysts who perform in-depth security reviews of applications. They will use the agent to accelerate their initial triage process, quickly identifying high-risk components and understanding potential attack paths. The interactive chatbot will allow them to dig deeper into specific vulnerabilities or components without re-running a full scan.

  - Security Analysts (SOC/Threat Intelligence): Analysts who may receive SBOMs from internal development teams or third-party vendors. They will use the agent to rapidly assess the threat landscape associated with a given piece of software and use the chatbot to ask clarifying questions about the findings.

- Secondary Audience:

  - Security-Conscious Software Developers: Developers who want to take a more proactive role in securing their code. The agent provides them with a simple way to check their dependencies for critical issues before committing code.

  - Product Security Managers: Managers who need high-level summaries of the risk posture of different applications. The agent's clear, file-based Markdown reports can be easily shared and archived to track risk over time.

## 2.0 User Personas and Stories

### 2.1 Primary Persona: "Alex," the DevSecOps Engineer

- Profile: Alex is responsible for building and maintaining the secure software development lifecycle (SDLC) at a mid-sized technology company. They are highly technical, fluent in Python and shell scripting, and live in the command line. Alex's primary challenge is to balance the speed of development with the need for robust security. They are often inundated with a high volume of alerts from various security scanners and struggle to distinguish real, urgent threats from theoretical or low-risk findings.

- Goals:

  - To quickly and accurately assess the security risk of new dependencies, incorporating predictive data like EPSS to better gauge future threats.

  - To understand the real-world impact of a vulnerability—is it being actively exploited by threat actors?—rather than relying solely on its theoretical CVSS severity score.

  - To provide development teams with clear, unambiguous, and actionable guidance on which vulnerabilities to fix first, supported by an AI-generated summary of the key risks.

  - To automate security analysis as a gate within the CI/CD pipeline, failing a build if a high-risk, actively exploited vulnerability is detected, and saving the report artifact for review.

  - To automatically store each processed SBOM in a local database, so they can reference it later and track changes over time.

  - To be able to ask follow-up questions about a scan using a natural language chatbot, such as "Show me all critical vulnerabilities in the 'acme-library' component" or "What is the highest priority vulnerability and why?".

  - To have clear instructions and a simple command to set up the required database locally using Docker.

  - **To leverage a RAG vector store for more accurate and contextualized LLM responses, especially for defensive measures and CVE information.**

  - **To ensure the agent can process very large SBOMs without hitting LLM token limits.**

### 2.2 User Stories

The following user stories describe the key tasks Alex needs to accomplish using the Cyber Threat Intelligence Agent:

- US-01: SBOM Analysis Initiation
  - "As Alex, I want to run the agent from my command line, passing the path to a CycloneDX SBOM file, so that I can initiate a security analysis of my application's dependencies."

- US-02: Prioritized Vulnerability Reporting
  - "As Alex, I want to receive a clear, prioritized list of vulnerabilities, so I can immediately focus my team's efforts on the highest-risk issues."

- US-03: AI-Driven CPE Identification
  - "As Alex, I want the agent to automatically identify the correct Common Platform Enumeration (CPE) for each software component, even if the SBOM does not contain it, so that it can accurately query for associated CVEs."

- US-04: Threat Contextualization
  - "As Alex, I want to see how each critical vulnerability maps to specific MITRE ATT&CK techniques and Common Weakness Enumerations (CWEs), so I can understand the potential attack vectors."

- US-05: Actionable Defensive Measures
  - "As Alex, I want the agent to provide me with any available Sigma, Snort, or Yara rules for high-priority vulnerabilities, so I can immediately provide them to our Security Operations team."

- US-06: CISA KEV Correlation
  - "As Alex, I want the agent to clearly flag any vulnerabilities that are listed in the CISA KEV catalog, so I can give them the highest possible priority."

- US-07: Secure Configuration
  - "As Alex, I want to configure the agent with my necessary API keys using a standard .env file, so that my sensitive credentials are not exposed."

- US-08: SBOM Persistence
  - "As Alex, I want the agent to automatically store each processed SBOM in a local database, so I can reference it later and track changes over time."

- US-09: Historical SBOM Retrieval
  - "As Alex, I want to be able to list and retrieve previously analyzed SBOMs from the database, so I can re-run analysis or review past reports."

- US-10: Local Database Setup
  - "As Alex, I want clear instructions and a simple command to set up the required database locally using Docker."

- US-11: Enhanced Prioritization with EPSS
  - "As Alex, I want the agent to incorporate the Exploit Prediction Scoring System (EPSS) score into its risk analysis, so I can better prioritize vulnerabilities that are likely to be exploited in the near future."

- US-12: AI-Generated Summary
  - "As Alex, I want the final report to include a 1-2 paragraph, AI-generated executive summary, so I can quickly understand the key findings and share them with stakeholders."

- US-13: Interactive Chatbot Query
  - "As Alex, after a scan is complete, I want to be able to enter an interactive 'chat' mode, so I can ask natural language questions about the results (e.g., 'Which components have actively exploited vulnerabilities?') and get immediate answers without running a new scan."

- US-14: File-Based Reporting
  - "As Alex, I want the agent to save the final Markdown report to a file in a designated reports/ directory, so I can easily archive it, share it, or use it in my CI/CD pipeline artifacts."

- **US-15: Enhanced Defensive Measures**: "As Alex, I want the agent to provide more comprehensive defensive measures, including mitigation/remediation/patching instructions, by leveraging LLMs and a RAG vector store."

- **US-16: Contextualized CVE Information**: "As Alex, I want the LLM to provide richer context for CVEs by scraping NVD and storing information in a RAG vector store."

- **US-17: Large SBOM Handling**: "As Alex, I want the agent to efficiently analyze large SBOMs without encountering LLM token limits."

- **US-18: Local Vector Store Deployment**: "As Alex, I want clear instructions and a simple command to set up a local vector store for RAG."

## 3.0 Functional Requirements: Core Features

This section details the specific functional capabilities that the Cyber Threat Intelligence Agent must provide.

### 3.1 Command-Line Interface (CLI)

- FR-3.1.1: SBOM Input: The CLI MUST accept a mandatory argument to specify the path to the CycloneDX SBOM file for analysis (`--sbom-file <path>`).
- FR-3.1.2: Pydantic AI Integration: The CLI will continue to be implemented using the Pydantic AI framework's `to_cli()` or `to_cli_sync()` methods.
- FR-3.1.3: Output to File: The CLI MUST save its final analysis report as a well-formatted Markdown file to a `./reports/` directory. The filename should be derived from the input SBOM filename and a timestamp to ensure uniqueness (e.g., `report-log4j-sbom-20250721103000.md`). It will no longer print the full report to standard output, but will instead print the path to the generated report file.
- FR-3.1.4: Error Handling: The application must maintain robust error handling for scenarios such as invalid file paths, missing API keys, malformed SBOMs, or API failures.
- FR-3.1.5: Interactive Chat Mode: The CLI MUST include a new command, such as `agent chat`, that launches an interactive session. In this mode, the user can ask natural language questions, and the agent will query the MongoDB database to provide answers based on previously ingested SBOMs.

### 3.2 CycloneDX SBOM Parsing

(No changes from v3.0)

### 3.3 Multi-Source Vulnerability Correlation

- FR-3.3.1: NVD API Integration: The agent MUST query the NVD 2.0 CVE API.
- FR-3.3.2: CISA KEV Catalog Integration: The agent MUST correlate identified CVEs against the CISA KEV catalog.
- FR-3.3.3: EPSS Integration: The agent MUST retrieve the Exploit Prediction Scoring System (EPSS) score for each identified CVE. This will require integrating with the official EPSS API or a library that provides this data. The EPSS score will be a key input into the prioritization algorithm.

### 3.4 AI-Powered Analysis & Contextualization

- FR-3.4.1: AI-Driven CPE Construction: The agent MUST use an LLM to construct valid CPE strings for components where they are missing.
- FR-3.4.2: Threat Contextualization (CWE & MITRE ATT&CK): The agent MUST use an LLM to map CVEs to CWEs and MITRE ATT&CK techniques.
- FR-3.4.3: Defensive Measure Analysis: The agent MUST use an LLM to find relevant defensive measures (Sigma, Snort, Yara).
- FR-3.4.4: AI-Generated Executive Summary: The agent MUST use the LLM at the end of the analysis to generate a 1-2 paragraph executive summary of the findings. This summary will be placed at the top of the Markdown report.
- **FR-3.4.5: Enhanced Defensive Measures**: The agent MUST leverage LLMs and a RAG vector store to provide comprehensive defensive measures, including mitigation, remediation, and patching instructions.
- **FR-3.4.6: Contextualized CVE Information**: The agent MUST use LLMs to scrape NVD for detailed CVE information and store it in a RAG vector store to provide richer context during analysis.
- **FR-3.4.7: Large SBOM Handling**: The agent MUST implement strategies (e.g., chunking, summarization) to handle large SBOMs and associated vulnerability data to avoid LLM token limits.

### 3.5 Comprehensive Reporting

- FR-3.5.1: Report Format: The final output MUST be a single, comprehensive report generated in Markdown format and saved to a file.
- FR-3.5.2: Report Structure: The report structure will be updated to include the new AI-generated summary and EPSS scores.
    1. **AI-Generated Executive Summary**: A 1-2 paragraph overview of the key findings.
    2. Prioritized Vulnerabilities: A summary table ordered by the `RiskScore`.
    3. Detailed Vulnerability Analysis: A section for each high-priority vulnerability, now including:
       - CVE ID, CVSS Score, CISA KEV Status, CWE, MITRE ATT&CK Mapping.
       - **EPSS Score**: The EPSS probability score.
       - Defensive Measures.
- FR-3.5.3: AI-Powered Prioritization: The report MUST present vulnerabilities in a prioritized order determined by the agent's internal risk-scoring algorithm, which will now include the EPSS score.

### 3.6 Database Integration (MongoDB via Docker)

- FR-3.6.1: Database Choice: The agent MUST integrate with MongoDB as its primary data store.
- FR-3.6.2: Local Deployment: The project MUST provide clear instructions for deploying a local MongoDB instance using Docker.
- FR-3.6.3: Connection Management: The Python application MUST connect to MongoDB using details from environment variables.
- FR-3.6.4: SBOM Ingestion: The agent MUST store the raw SBOM content in MongoDB after parsing.
- FR-3.6.5: SBOM Deduplication: The agent MUST prevent the storage of identical SBOMs. If an SBOM with the same content already exists in the database, it MUST be identified as a duplicate, and the existing data should be used for analysis instead of re-ingesting.
- FR-3.6.6: SBOM Querying for Chatbot: The database integration MUST support the new interactive chat feature by enabling efficient querying of the stored SBOM data based on natural language questions translated into database queries by the LLM.

### 3.7 Data Model (MongoDB Document Structure)

(No changes from v3.0)

### 3.8 Vector Store Integration

- **FR-3.8.1: Vector Store Choice**: The agent MUST integrate with a local vector store (e.g., ChromaDB, FAISS) for RAG capabilities.
- **FR-3.8.2: Local Deployment**: The project MUST provide clear instructions for deploying and initializing the local vector store.
- **FR-3.8.3: Data Ingestion**: The agent MUST be able to ingest scraped NVD data and defensive measures into the vector store.
- **FR-3.8.4: Vector Search**: The agent MUST be able to perform vector similarity searches to retrieve relevant context for LLM queries.

## 4.0 AI Agent Architecture

### 4.1 Pydantic AI Agent Definition

(No changes to initialization, model pluggability, or system prompt from v3.0)

### 4.2 Agent Tool Definitions

The tool definitions will be expanded to support the new features.

| Tool Name | Function | Inputs | Outputs | Example LLM Thought Process |
| --- | --- | --- | --- | --- |
| ... (existing tools) | ... | ... | ... | ... |
| **query_epss** | Queries the EPSS API for the exploit probability score of a given CVE. | `cve_id: str` | `epss_score: float` | "I have found CVE-2023-1234. To better assess its risk, I need to know how likely it is to be exploited. I will call the `query_epss` tool with this CVE ID." |
| **summarize_findings** | Takes all enriched vulnerability data and generates a high-level executive summary. | `enriched_vulnerabilities: List[dict]` | `summary_text: str` | "The analysis is complete. I need to create a summary for the report. I will call `summarize_findings` with the full list of vulnerabilities to generate a concise overview." |
| **answer_user_query** | (For Chat Mode) Queries the MongoDB database based on a user's natural language question. | `user_question: str` | `answer_text: str` | "The user is in chat mode and asked 'Which components are from Apache?'. I need to translate this into a database query and return the answer. I will call `answer_user_query` with the user's question." |
| **scrape_nvd_cve_info** | Scrapes detailed information for a given CVE from NVD and stores it in the RAG vector store. | `cve_id: str` | `success: bool` | "I need more detailed context for CVE-2023-1234. I will call `scrape_nvd_cve_info` to get the full NVD page content and store it." |
| **search_defensive_measures** | Searches the RAG vector store for defensive measures (Sigma, Snort, Yara, Mitigation/Remediation/Patching Instructions) related to a CVE. | `cve_id: str` | `defensive_measures: List[str]` | "I have identified a high-risk CVE. I need to find relevant defensive measures. I will call `search_defensive_measures` with the CVE ID." |
| **handle_large_sbom_chunk** | Processes a chunk of a large SBOM to extract components and associated data, managing token limits. | `sbom_chunk: str` | `components: List[dict]` | "The SBOM is too large for a single LLM call. I will use `handle_large_sbom_chunk` to process it in smaller parts." |

### 4.3 Orchestration Logic

The workflow will be updated to include the new steps:

1. Parse SBOM (potentially in chunks for large SBOMs).
2. For each component:
   - Get CPE if missing.
   - Query NVD for CVEs.
   - If CVEs are found:
     - Correlate with CISA KEV.
     - **Query EPSS for exploitability score.**
     - Map to ATT&CK.
     - **Scrape NVD for detailed CVE info and ingest into RAG vector store.**
     - **Search RAG vector store for comprehensive defensive measures (Sigma, Snort, Yara, Mitigation/Remediation/Patching Instructions).**
3. After all components are processed, aggregate the data.
4. Apply the updated prioritization algorithm.
5. **Call `summarize_findings` to generate the executive summary.**
6. Synthesize all information into the final Markdown report and save it to a file.

### 4.4 AI-Driven Vulnerability Prioritization Algorithm

- FR-4.4.1: Multi-Factor Risk Scoring: The agent MUST calculate a proprietary `RiskScore` for each identified vulnerability.
- FR-4.4.2: Scoring Factors: The algorithm will be updated to include EPSS.
  - Severity (CVSS)
  - Exploitation Status (CISA KEV)
  - **Predicted Exploitability (EPSS)**: The EPSS score will be a significant factor, providing a forward-looking view of risk.
  - Threat Context (ATT&CK)

The updated formula will be:
`RiskScore = (W_cvss * CVSS) + (W_kev * KEV) + (W_epss * EPSS) + (W_attack * ATTACK)`

## 5.0 Data Management & Integration

### 5.1 External Data Sources

The table of external data sources will be updated to include EPSS.

| Data Source | Format | Acquisition Method | URL / Endpoint | Local Path (frameworks/) | Update Strategy |
| --- | --- | --- | --- | --- | --- |
| NVD | JSON | Live API Call | `https://services.nvd.nist.gov/rest/json/cves/2.0` | N/A | Real-time per run |
| CISA KEV | JSON | Download | `https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json` | cisa_kev.json | Daily |
| **EPSS** | JSON | Live API Call | `https://api.first.org/epss/v3/cve/{cve_id}` | N/A | Real-time per run |
| MITRE ATT&CK | STIX 2.1 | Download | `https://github.com/mitre/cti/raw/master/enterprise-attack/enterprise-attack.json` | enterprise-attack.json | Quarterly |
| CWE | XML | Download (ZIP) | `https://cwe.mitre.org/data/xml/cwec_v4.14.xml.zip` | cwe_database.xml | Annually |
| **NVD Website** | HTML | Web Scraping | `https://nvd.nist.gov/vuln/detail/{cve_id}` | N/A | On-demand/Cached |

### 5.2 Internal Data Models (Pydantic)

The `EnrichedVulnerability` model will be updated to include the EPSS score.

- `EnrichedVulnerability`: Will now include `epss_score: Optional[float]`.

## 6.0 Non-Functional Requirements

(No changes from v3.0)

## 7.0 Project Deliverables & Setup

### 7.1 Project Directory Structure

The directory structure will be updated to include a `reports` directory.

```bash
\ cyber-threat-agent
|--.env.example
|--.gitignore
|-- README.md
|-- pyproject.toml
|-- frameworks/
|-- reports/
| |--.gitkeep
|-- src/
| |-- cti_agent/
| | |-- ... (existing files)
|-- tests/
|-- (Unit and integration tests)
|-- vector_store/ # New directory for local vector store
```

### 7.2 README.md Content Requirements

The README will be updated to reflect the new features:

- Mention the inclusion of EPSS scores.
- Describe the new interactive `chat` mode.
- Explain that reports are now saved to the `reports/` directory.
- **Add instructions for setting up and managing the local vector store.**

### 7.3 .env.example Specification

(No changes from v3.0)

### 7.4 Dependency Management with uv

(No changes from v3.0)

## 8.0 Assumptions and Out of Scope

### 8.1 Assumptions

(No changes from v3.0)

### 8.2 Out of Scope for Version 4.0

- Graphical User Interface (GUI): This remains a CLI-first tool.
- Automated Remediation: The agent provides intelligence but does not perform automated actions.
- Real-time Application Monitoring: The agent operates on static SBOM files.
- Support for Other SBOM Formats: Support remains limited to CycloneDX.
- Advanced Historical Analysis Features: While the chatbot allows for basic querying, advanced features like trend analysis or diffing SBOMs over time are out of scope.
- **Complex, dynamic queries or aggregations across datasets beyond what the RAG and LLM can handle with current tools.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PrymalInstynct)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/PrymalInstynct)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
