## keycloak-push-mfa-extension

> - Always finish a task by running `mvn spotless:apply` and `mvn verify` so formatting and tests stay green.

# AGENTS NOTES

- Always finish a task by running `mvn spotless:apply` and `mvn verify` so formatting and tests stay green.
- The example realm JSON lives at `config/demo-realm.json` and defines the realm `demo`; helper scripts and docs should reference `/realms/demo/...`.
- The build outputs a fixed artifact `target/keycloak-push-mfa-extension.jar`; docker-compose and ITs depend on that filename.
- Device-facing endpoints live under `/realms/<realm>/push-mfa/...` and expect DPoP-bound tokens; keep samples and tests aligned with the current realm name and URL structure.
- If java does not seem to be installed, check sdkman

---
> Source: [ba-itsys/keycloak-push-mfa-extension](https://github.com/ba-itsys/keycloak-push-mfa-extension) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
