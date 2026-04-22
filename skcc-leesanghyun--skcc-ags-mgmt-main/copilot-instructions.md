## skcc-ags-mgmt-main

> Apply these rules when making changes to the project


Update this rule if user requested changes to the project requirement, etc.
flowchart TD
    A[Start] --> B[Login - OAuth2 Okta integration]
    B --> C{User Role?}
    C -->|PM| D1[Project Registration and SOW Management]
    D1 --> E1[Candidate Sourcing and Evaluation]
    E1 --> F1[SOW Download]
    C -->|협력사 Manager| D2[Proposal Creation and Feedback]
    D2 --> E2[Staffing Recommendations]
    C -->|Business Team| D3[Access Planning and Performance Dashboard]
    C -->|Delivery Innovation Team| D4[Onboarding Offboarding Audit]
    D4 --> E4[서약서 Upload and Management]
    C -->|Information Security Team| D5[Review Security and Audit Data]
    D5 --> E5[서약서 Compliance Check]
    C -->|System Administrator| D6[System Administration and Deployment Management]
    F1 --> G[Persist Data in Single PostgreSQL Instance]
    E2 --> G
    E4 --> G
    E5 --> G
    D3 --> G
    G --> H[Internationalization and UI Management]
    H --> I[Modern Corporate Dashboard]
    I --> J[Logging and Monitoring]
    J --> K[End]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skcc-leesanghyun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
