## curated-content-portal

> **PRIMARY SOURCE**: `docs/BUILD_AND_DEPLOYMENT_PROCESS.md`

# Clean Portal - AI Agent Rules (Condensed)
# Version: 2.0 | Updated: 2026-01-23 | Tokens: ~1K

---

## 🚨 DEPLOYMENT AUTHORITY (IMMUTABLE)

**PRIMARY SOURCE**: `docs/BUILD_AND_DEPLOYMENT_PROCESS.md`

**RULE**: Read entire doc BEFORE any deployment. NO deviations without explicit user approval.

**Violation Consequences**: Deployment failures, service outages, config drift, loss of trust.

---

## 📋 AGENT STARTUP (REQUIRED)

```bash
# 1. Read this file (you are here)
# 2. Read docs/AGENT_REFERENCE.md (condensed reference, ~2K tokens)
# 3. Run health check
./scripts/health-check-consolidated.sh remote
# 4. If issues found
./scripts/health-check-consolidated.sh remote --fix
# 5. If SSH fails, server is down - STOP, user must start EC2
```

---

## 🎯 CRITICAL RULES

| Rule | Details |
|------|---------|
| **Remote-First** | dev.corsoro.com (13.219.45.187) is PRIMARY. Local is optional. |
| **Health Check** | `./scripts/health-check-consolidated.sh remote` ALWAYS. NEVER `curl localhost`. |
| **PM2** | NEVER `sudo pm2`. ALWAYS `pm2 restart app --update-env` after env changes. |
| **Environment** | `/opt/cleanportal/.env.dev` is stateful (NOT synced). Edit on server + restart PM2. |
| **Deployment** | Read BUILD_AND_DEPLOYMENT_PROCESS.md FIRST. Push → GH Actions → test on dev.corsoro.com. |
| **Logs** | User errors are generic. Check `pm2 logs app --err --lines 50` for REAL errors. |
| **Local Trap** | User says "dev.corsoro.com"? Check REMOTE. Don't start local Docker. |

---

## 🚫 ANTI-PATTERNS

| ❌ NEVER | ✅ ALWAYS |
|---------|-----------|
| `npm run dev` or `npm start` (NO LOCALHOST SERVER EXISTS) | Check/restart remote: `ssh cleanportal-dev 'pm2 restart app --update-env'` |
| `curl http://localhost:3000/api/health` | `./scripts/health-check-consolidated.sh remote` |
| Start localhost development server | Development server IS dev.corsoro.com (already running) |
| Check localhost for server issues | Check remote server |
| `sudo pm2 restart app` | `pm2 restart app` (no sudo) |
| Make up commands | Use provided scripts |
| Start local Docker for server issues | Check remote first |
| "Start dev server" → localhost | "Start dev server" → check/restart remote PM2 |

---

## 🔧 SERVER

```
Dev:  ssh cleanportal-dev (13.219.45.187, ubuntu, .env.dev)
Prod: ssh cleanportal-prod (44.198.212.206, ec2-user, .env.production)
Dir:  /opt/cleanportal (both servers)
```

---

## 🚀 WORKFLOW

```
Edit code → Commit → Push main → GH Actions deploys (5-10 min) → Test on dev.corsoro.com
```

**Deploy Process**: rsync → npm ci → build → PM2 reload → health check

**Manual Deploy** (if GH Actions fails):
```bash
ssh cleanportal-dev
cd /opt/cleanportal && git pull && npm ci && npm run build && pm2 reload app --update-env
```

---

## 🏥 TROUBLESHOOTING

| Issue | Command |
|-------|---------|
| PM2 errored | `./scripts/health-check-consolidated.sh remote --fix` |
| 502 errors | `ssh cleanportal-dev 'pm2 logs app --err --lines 50'` |
| Works locally not server | Check `/opt/cleanportal/.env.dev`, restart PM2 |
| SSH timeout | Server down - user must start EC2 |

---

## 📚 DOCUMENTATION

**Main Reference**: `docs/AGENT_REFERENCE.md` (~2K tokens)

**Deep Dives**:
- Deployment: `docs/BUILD_AND_DEPLOYMENT_PROCESS.md` (REQUIRED before deploy)
- Debugging: `docs/LESSONS_DIAGNOSTIC_METHODOLOGY.md`
- PM2 Issues: `docs/ZOMBIE_PROCESS_INCIDENT_2026-01-06.md`

**Archived**: `archive/agent-docs-2026-01-23/` (old 60KB verbose docs)

---

## 🔗 QUICK COMMANDS

```bash
# Health check (ALWAYS start here)
./scripts/health-check-consolidated.sh remote

# SSH
ssh cleanportal-dev

# Logs
ssh cleanportal-dev 'pm2 logs app --err --lines 50'

# Restart
ssh cleanportal-dev 'pm2 restart app --update-env'

# Deploy status
gh run list --limit 5
```

---

## 💡 REMEMBER

- ⚠️ **NO LOCALHOST SERVER**: This project has NO localhost development server. Never run `npm run dev`.
- ✅ **"Start dev server"** = Check/restart remote PM2 at dev.corsoro.com
- ✅ Read `docs/AGENT_REFERENCE.md` (replaces 4 verbose docs)
- ✅ Use scripts, don't make up commands
- ✅ Check remote by default (dev.corsoro.com)
- ✅ Never `sudo pm2`
- ✅ Trust code over assumptions
- ✅ Read BUILD_AND_DEPLOYMENT_PROCESS.md before ANY deploy

---

**BMAD Integration**: Available via `@bmad/` references (see `.cursor/rules/bmad/index.mdc`)

**Version**: 2.0-condensed | **Tokens**: ~1K (was 7.6K) | **Backup**: `archive/agent-docs-2026-01-23/cursorrules.backup`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jcaisse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
