---
name: apollo-io
description: Access sales intelligence and engagement via Apollo.io API. Find leads and manage sequences. Use when this capability is needed.
metadata:
  author: openclaw
---
# Apollo.io
Sales intelligence platform.
## Environment
```bash
export APOLLO_API_KEY="xxxxxxxxxx"
```
## Search People
```bash
curl -X POST "https://api.apollo.io/v1/mixed_people/search" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "'$APOLLO_API_KEY'", "person_titles": ["CEO", "CTO"], "organization_num_employees_ranges": ["1,50"]}'
```
## Search Organizations
```bash
curl -X POST "https://api.apollo.io/v1/mixed_companies/search" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "'$APOLLO_API_KEY'", "organization_num_employees_ranges": ["1,50"]}'
```
## Enrich Person
```bash
curl -X POST "https://api.apollo.io/v1/people/match" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "'$APOLLO_API_KEY'", "email": "ceo@example.com"}'
```
## Create Contact
```bash
curl -X POST "https://api.apollo.io/v1/contacts" \
  -H "Content-Type: application/json" \
  -d '{"api_key": "'$APOLLO_API_KEY'", "first_name": "John", "last_name": "Doe", "email": "john@example.com"}'
```
## Links
- Dashboard: https://app.apollo.io
- Docs: https://apolloio.github.io/apollo-api-docs/

---
> Converted and distributed by [TomeVault](https://tomevault.io) | [Claim this content](https://tomevault.io/claim/openclaw/skills)
<!-- tomevault:3.0:skill_md:2026-04-07 -->
