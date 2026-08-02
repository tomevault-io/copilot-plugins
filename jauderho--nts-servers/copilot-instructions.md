## nts-servers

> - Scan the URL provided or GitHub Issue for any relevant information

# AGENTS.md

- Scan the URL provided or GitHub Issue for any relevant information
- Extract hostname and verify that the NTS server is reachable via NTS and NTP using ./scripts/ntsCheck.sh
- Place new entry in the proper location in nts-sources.yml by alphabetical country
- Do not make direct changes to README.md, chrony.conf and ntp.toml. Run ./scripts/ntpServerConvertor.py to update after changes to nts-sources.yml are completed

---
> Source: [jauderho/nts-servers](https://github.com/jauderho/nts-servers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
