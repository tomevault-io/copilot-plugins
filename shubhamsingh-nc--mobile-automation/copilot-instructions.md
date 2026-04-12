## mobile-automation

> Mobile E2E test automation using [Maestro](https://docs.maestro.dev/) for NymCard apps. YAML flows + JavaScript POM.

# CLAUDE.md

Mobile E2E test automation using [Maestro](https://docs.maestro.dev/) for NymCard apps. YAML flows + JavaScript POM.

## Commands

```bash
maestro test <client>/.maestro/                              # Run all tests
maestro test <client>/.maestro/<feature>/Flow.yaml           # Run specific flow
maestro test --include-tags=smoke <client>/.maestro/         # Run by tag
maestro test --format junit --output report.xml <client>/.maestro/  # JUnit report
```

Run from project root.

## Structure

```
<client>/.maestro/
├── config.yaml          # Flow patterns, output dir
├── elements/*.js        # POM definitions
└── <feature>/*.yaml     # Test flows
```

## POM Pattern

```javascript
// elements/login.js
output.login = { phoneInput: 'phone_input', loginBtn: 'login_btn' }
```

```yaml
# flows/auth/Login.yaml
- runScript: { file: ../elements/login.js }
- tapOn: { id: ${output.login.phoneInput} }
```

## Conventions

- Clients: `lowercase` (ata)
- Features: `lowercase` (auth, cards)
- Flows: `PascalCase` (LoginSuccess.yaml)
- Elements: `lowercase` (login.js)

## Clients

- `ata` - Reference implementation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shubhamsingh-nc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shubhamsingh-nc)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
