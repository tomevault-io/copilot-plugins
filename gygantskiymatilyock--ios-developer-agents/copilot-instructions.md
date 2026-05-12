## ios-developer-agents

> This repository contains AI agent prompts for iOS development workflows. These agents help validate, audit, and improve iOS applications.

# iOS Developer Agents

This repository contains AI agent prompts for iOS development workflows. These agents help validate, audit, and improve iOS applications.

## Available Agents

### App Store Validator
Pre-validates iOS apps against Apple's App Store Review Guidelines before submission. Catches rejection-worthy issues early.

**Use when:** Preparing to submit to App Store Connect

### Security Auditor
Comprehensive security audit against OWASP Mobile Top 10 2024. Detects vulnerabilities, hardcoded secrets, and insecure configurations.

**Use when:** Before release, after adding authentication, or during security reviews

### Accessibility Validator
Audits apps for accessibility compliance with Apple HIG, WCAG 2.2, VoiceOver, and Dynamic Type standards.

**Use when:** Building UI, before releases, or when improving accessibility

## Usage with Claude Code

Use the slash commands to invoke agents:

- `/validate-appstore` - Run App Store validation on the current project
- `/audit-security` - Run security audit on the current project
- `/audit-accessibility` - Run accessibility audit on the current project

## Agent Files

Source prompts are in `agents/[name]/[name].md`:
- `agents/app-store-validator/app-store-validator.md`
- `agents/security-auditor/security-auditor.md`
- `agents/accessibility-validator/accessibility-validator.md`

## For iOS Projects Using These Agents

When using these agents on your iOS project, provide context about:
- Core app functionality
- Authentication methods
- Third-party SDKs used
- Sensitive data handled
- Compliance requirements (HIPAA, PCI-DSS, etc.)

---
> Source: [gygantskiyMatilyock/ios-developer-agents](https://github.com/gygantskiyMatilyock/ios-developer-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
