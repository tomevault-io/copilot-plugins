## sensitive-data

> Prevent leaking sensitive customer and infrastructure data in PR descriptions, commits, and external-facing content


# Sensitive Data Policy

This repository contains real customer names, cluster names, and infrastructure details (region names, pop locations, etc.) in its configuration files and CLI output.

**Never include real customer or infrastructure data in:**
- PR descriptions or commit messages
- Code comments explaining examples
- Any content that may be visible on GitHub or other external platforms

**When showing CLI output examples**, use generic placeholders:
- Regions: `region1`, `region2`, `us-east`, `eu-west`
- Clusters: `cluster-a`, `cluster-b`, `default`
- Customers: `customer1`, `customer2`

---
> Source: [getsentry/sentry-infra-tools](https://github.com/getsentry/sentry-infra-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
