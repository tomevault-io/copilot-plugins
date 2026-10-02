## saml-research

> This project implements a pipeline for SAML bypasses. The structure is as

# SAML hacking pipeline

This project implements a pipeline for SAML bypasses. The structure is as
follows:

- ./api - holds the JSON schemas for all jsonl files.
- ./investigate/{repo-slug} - holds investigations for individual projects. Sub
  directories include:
  - ./vendor - source code of the project
  - ./artifacts - artifact directories for findings and gadgets
  - ./surface.jsonl - API surface for the SAML validation logic
  - ./gadgets.jsonl - List of primitives for causing unexpected behavior in a
    library.
  - ./findings.jsonl - End-to-end findings for a project
- ./knowledge - contains files that are shared between agents.
  - ./analyze.md - instructions for what parts of the codebase are worth
    inspecting.
  - ./scope.md - what this pipeline consider "in-scope" and importantly, what's
    "out-of-scope". Any end-to-end finding should adhere to this.
  - ./xmldsig.md - a primer on XMLDSig. Useful for any agent that wants to
    understand what to look for.

## General tips

- Use the `xmlsec1` and `openssl` command line tools to generate test cases.

---
> Source: [oblique-security/saml-research](https://github.com/oblique-security/saml-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
