## singbox-grafana-ksu

> This project builds an online KernelSU module for an Android ARM64 monitoring

# Project rules

This project builds an online KernelSU module for an Android ARM64 monitoring
stack. The stack is Grafana, Prometheus, and the signed Infinity datasource
plugin. Keep the module loopback-only by default.

## Security

- Never execute downloaded content before its SHA-256 has been checked.
- Keep runtime data and tokens under the root-owned
  /data/adb/singbox-observability directory.
- Do not put a real sing-box token, Grafana password, or generated secret in
  this repository.
- The Infinity plugin archive must retain its Grafana-signed MANIFEST.txt.
- Do not expose either service outside 127.0.0.1 without a deliberate TLS and
  authentication design review.

## Runtime contracts

- sing-box remains on 127.0.0.1:9090; its Bearer token comes from persistent
  settings.
- Prometheus listens on 127.0.0.1:9091 by default.
- Grafana listens on 127.0.0.1:3000 by default.
- Prometheus retention always has both a time and size limit.
- Grafana provisioning uses stable datasource UIDs singbox-prometheus and
  singbox-infinity.
- Generate the provisioned dashboard from dashboard-source.json; do not
  hand-edit generated dashboard files.

## Validation

Run these commands before shipping:

~~~sh
./scripts/build.sh
./scripts/validate.sh
./scripts/test-config-generation.sh
./scripts/verify-upstream.sh
~~~

Checks must cover shell syntax, JSON validity, datasource references, pinned
hashes, and ZIP integrity. Never run Android service scripts on a workstation.

---
> Source: [CHIZI-0618/singbox-grafana-ksu](https://github.com/CHIZI-0618/singbox-grafana-ksu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
