## 3x-ui-pro

> Refactored single-file installer for 3x-ui VPN panel (based on x-ui-pro).

# x-ui-pro-refactor

Refactored single-file installer for 3x-ui VPN panel (based on x-ui-pro).

## Repository structure

```
x-ui-latest.sh          — main installer script (single file, run remotely)
assets/
  fake-sites/
    site-01/ … site-50/ — static HTML cover pages (index.html per site)
x-ui-installer/         — reference project (do not modify)
x-ui-pro/               — original project (do not modify)
```

## What the script does

1. Stops / cleans any previous install
2. Parses CLI arguments (`-install y`, `-subdomain`, `-reality_domain`, `-auto_domain y`, `-uninstall y`)
3. Installs packages (`install_packages`) — nginx-full, certbot, sqlite3, curl, wget, jq, ufw
4. Obtains Let's Encrypt certs via certbot standalone (`get_ssl_certs`) — two domains: panel + reality
5. Installs 3x-ui panel from MHSanaei/3x-ui latest release (`install_panel`)
6. Configures nginx (`configure_nginx`) — SNI stream, per-domain vhosts, shared includes snippet
7. Pushes all settings and inbounds into x-ui.db (`configure_xui_db`) — untouched from original
8. Downloads a random fake cover site from this repo (`install_fake_site`) → `/var/www/html/`
9. Tunes kernel/BBR (`tune_system`)
10. Sets up cron (`setup_cron`) — daily x-ui restart + nginx reload, monthly certbot renew
11. Configures UFW (`setup_firewall`) — 22/80/443 tcp, 443 udp
12. Prints panel URL + credentials (`show_results`)

## What was intentionally removed vs original

- `sub2singbox` binary install and `@reboot` cron entry
- Custom web subscription page (`/var/www/subpage`, clash.yaml, sub-3x-ui.html)
- `web_path` and `sub2singbox_path` nginx locations
- Call to external `randomfakehtml.sh` → replaced with `install_fake_site()` using local assets

## Inbounds created

| Protocol     | Port          | Transport |
|--------------|---------------|-----------|
| vless        | 8443          | REALITY / TCP |
| vless        | `$ws_port`    | WebSocket  |
| vless        | UDS socket    | XHTTP (gRPC) |
| trojan        | `$trojan_port`| gRPC       |

## GitHub raw URL for fake sites

`https://raw.githubusercontent.com/mozaroc/3x-ui-pro/main/assets/fake-sites/site-NN/index.html`

## Running

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/mozaroc/3x-ui-pro/main/x-ui-latest.sh) \
  -install y -subdomain panel.example.com -reality_domain r.example.com
```

---
> Source: [mozaroc/3x-ui-pro](https://github.com/mozaroc/3x-ui-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
