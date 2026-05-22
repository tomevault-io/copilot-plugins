## structify

> Structify—the goal behind this app is to allow people to define a table and its expected content.

# Structify

Structify—the goal behind this app is to allow people to define a table and its expected content.
Provide as much detailed information about the expected content as possible describing what does the table store.
This is because in the next steps the user will upload PDF files to an app from which this data will be extracted using
AI.
App integrates with OpenAI rest api. App has a React ui allowing to build tables, upload data, etc.
Building table is basically defining columns and the description of what is this column so the LLM can find this
information within provided documents.
App is multitenant, meaning different users should not be able to access another user's data.

# Backend

All backend related instructions are located in the file BACKEND.md.

# Frontend
Located under web/loveable. It is react ui generated with AI tool called loveable.

---
> Source: [damiankaplon/structify](https://github.com/damiankaplon/structify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
