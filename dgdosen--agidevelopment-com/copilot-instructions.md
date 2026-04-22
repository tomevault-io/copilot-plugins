## agidevelopment-com

> The root domain `agidevelopment.com` was resolving to unexpected third-party content.

# AGIDevelopment.com – DNS Cleanup & Static Site Setup

## Summary
The root domain `agidevelopment.com` was resolving to unexpected third-party content.
Domain ownership, registrar access, and DNS authority were never compromised.

The issue was resolved by removing unintended apex DNS records and confirming a clean baseline before attaching a new static site.

## DNS State at Discovery
The root domain (`@`) resolved via:

- A → `75.2.60.5`
- AAAA → `2607:7700:0:33:0:1:4b02:3c05`

Nameservers were correctly set to DigitalOcean:
- `ns1.digitalocean.com`
- `ns2.digitalocean.com`
- `ns3.digitalocean.com`

Google Workspace DNS records (MX, TXT, service CNAMEs) were present and correct.

## Resolution Steps
In DigitalOcean DNS:
- Removed the apex A record
- Removed the apex AAAA record
- Verified no wildcard (`*`) or apex CNAME records existed
- Left all Google Workspace records unchanged
- Left existing subdomain A/CNAME records unchanged

After TTL expiration, the root domain stopped resolving.

## Verification
Post-change verification commands:

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dgdosen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
