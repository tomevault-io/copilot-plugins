## homelab-horizon

> HTTP server, handlers, and HTML templates for the admin web UI.

# SERVER PACKAGE

HTTP server, handlers, and HTML templates for the admin web UI.

## STRUCTURE

```
server/
├── server.go              # Server struct, routes, middleware (1.2k lines)
├── handlers_*.go          # HTTP handlers grouped by domain
├── templates_*.go         # Raw HTML as Go strings (no template files)
└── (no subdirs)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add route | `server.go` → `setupRoutes()` | Match existing pattern |
| Add handler | `handlers_{domain}.go` | Create new file if new domain |
| Add template | `templates_{page}.go` | Raw HTML string constant |
| Modify admin UI | `templates_admin.go` | 1k+ lines, JS at bottom |
| Add template data | Handler's `data` map | Pass to `s.templates[name].Execute()` |

## CONVENTIONS

- **Handler naming**: `handle{Action}` (e.g., `handleAdmin`, `handleServiceEdit`)
- **Template data**: Always `map[string]interface{}` with `CSRFToken`
- **Redirects**: Use `?msg=` or `?err=` query params for flash messages
- **CSRF**: All POST require token; JS auto-injects from meta tag
- **Admin check**: `s.isAdmin(r)` or `s.requireAdminPost(w, r)`

## HANDLER FILES

| File | Domain |
|------|--------|
| `handlers_auth.go` | Login, logout, CSRF |
| `handlers_wireguard.go` | Client management, invites |
| `handlers_services.go` | Service CRUD, sync |
| `handlers_zones.go` | Zone management |
| `handlers_dns.go` | DNS provider discovery |
| `handlers_haproxy.go` | HAProxy status (mostly deprecated) |
| `handlers_ssl.go` | Certificate requests |
| `handlers_checks.go` | Health monitoring |
| `handlers_setup.go` | System config, setup wizard |

## TEMPLATE FILES

| File | Page |
|------|------|
| `templates_admin.go` | Main dashboard (1k lines) |
| `templates_setup.go` | Setup wizard |
| `templates_help.go` | Getting started guide |
| `templates_haproxy.go` | HAProxy/SSL management |
| `templates_dns.go` | External DNS records |
| `templates_checks.go` | Health check status |
| `templates_auth.go` | Login page |
| `templates_base.go` | Shared CSS (`baseCSS` constant) |
| `templates_wireguard.go` | WG client config display |

## ANTI-PATTERNS

- **Don't create template files**: All HTML lives in Go strings
- **Don't refactor templates_admin.go**: 1k lines is intentional
- **Deprecated handlers**: `handleRoute53*`, `handleHAProxyBackend*` → redirect to services

---
> Source: [IodeSystems/homelab-horizon](https://github.com/IodeSystems/homelab-horizon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
