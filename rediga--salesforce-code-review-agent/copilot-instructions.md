## salesforce-code-review-agent

> Salesforce security review for credentials, CRUD/FLS, guest users, and permission metadata


# Salesforce security

- No tokens, passwords, or private keys in source. Callouts use Named or External Credentials.
- User-facing reads/writes must enforce CRUD and FLS (`USER_MODE`, `stripInaccessible`, or `SECURITY_ENFORCED`).
- Flag `without sharing` on PII / Case / Account queries without a documented reason.
- Guest and Experience Cloud access must be least privilege.
- Flag Connected Apps / External Client Apps with overly broad OAuth scopes or weak refresh-token policies.
- New permission sets should not grant `Modify All Data`, `View All Data`, `Author Apex`, or `Manage Users` unless that is the persona.
- Do not update `User.Title`, `ManagerId`, `Name`, or org-specific role fields as a "fix".

---
> Source: [rediga/Salesforce-Code-Review-Agent](https://github.com/rediga/Salesforce-Code-Review-Agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
