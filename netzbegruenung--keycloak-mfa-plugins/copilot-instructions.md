## keycloak-mfa-plugins

> Use Conventional Commits: `type(scope): description`. Scope is optional.

# Commit messages

Use Conventional Commits: `type(scope): description`. Scope is optional.

- **type**: any standard Conventional Commits type, chosen normally based on the nature of the change.
- **scope**: the module a change is confined to, using these short names:

  | Module dir | Scope |
  |---|---|
  | `app-authenticator` | `app` |
  | `email-authenticator` | `email` |
  | `enforce-mfa` | `enforce-mfa` |
  | `sms-authenticator` | `sms` |
  | `trusted-device-authenticator` | `device` |
  | `mfa-dev-runner` | `dev-runner` |

  Omit the scope for changes spanning multiple modules or the whole repo, e.g. `chore: update Keycloak version to 26.7.0`.

---
> Source: [netzbegruenung/keycloak-mfa-plugins](https://github.com/netzbegruenung/keycloak-mfa-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-09 -->
