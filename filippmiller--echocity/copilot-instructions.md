## echocity

> 1. Read `.claude/deploy-instructions.md` for deployment details

# EchoCity - Project Instructions

## Session Start Protocol

1. Read `.claude/deploy-instructions.md` for deployment details
2. Read this full `CLAUDE.md` for project rules and conventions
3. Report to user: "I've read the deployment instructions and all important notices for **echocity**. Deploy target: https://echocity.vsedomatut.com. Last verified: 2026-04-09. Build status: OK. WARNING: root URL returns 404."
4. Run `git status` and report current branch and any uncommitted changes

---

## Deployment

**Production**: Laptop server via Coolify (vsedomatut.com)

| Key | Value |
|-----|-------|
| URL | https://echocity.vsedomatut.com |
| Coolify UUID | ki5yt1xyoo1lgsbp5lv39p96 |
| Coolify API | https://coolify.vsedomatut.com/api/v1 |
| Coolify Token | `4\|ePUn53pwokOoe9Z89kZHCc2HDurHbvFW5sq6uhREe396c13d` |
| Build pack | dockerfile (`/Dockerfile`) |
| Git repo | git@github.com:filippmiller/echocity.git (main) |
| DB UUID | b13rk5k1ix7mckiqotydobja (separate Coolify resource) |

### How to Deploy

```bash
# Trigger deploy via Coolify API
curl -s -H "Authorization: Bearer 4|ePUn53pwokOoe9Z89kZHCc2HDurHbvFW5sq6uhREe396c13d" \
  "https://coolify.vsedomatut.com/api/v1/deploy?uuid=ki5yt1xyoo1lgsbp5lv39p96&force=true"
```

Coolify auto-deploys on push to `main`. Manual deploy via API if needed.

### Architecture

- **App**: Next.js (Dockerfile build)
- **Database**: PostgreSQL (separate Coolify resource, UUID: b13rk5k1ix7mckiqotydobja)
- **Migrations**: Prisma

### SSH Access to Server

```bash
ssh -o ProxyCommand="cloudflared access ssh --hostname ssh.vsedomatut.com" filip@ssh.vsedomatut.com
```

Container names use Coolify UUID pattern: `ki5yt1xyoo1lgsbp5lv39p96-*`

### Database

- **Type**: PostgreSQL
- **User**: echocity
- **DB**: echocity
- **Migrations**: Prisma
- **Container**: `b13rk5k1ix7mckiqotydobja` (stable name, separate resource)
- Access: `docker exec b13rk5k1ix7mckiqotydobja psql -U echocity -d echocity`

### Known Issues

- **Failed Prisma migrations**: If the app won't start with `Error: P3009 - migrate found failed migrations`, check `_prisma_migrations` table:
  ```sql
  SELECT migration_name, finished_at FROM _prisma_migrations WHERE finished_at IS NULL;
  ```
  Fix by marking as complete:
  ```sql
  UPDATE _prisma_migrations SET finished_at = NOW(), rolled_back_at = NULL, logs = NULL WHERE finished_at IS NULL;
  ```
  Then restart the app. **Note**: This marks migrations as done without actually running them — ensure the schema changes were already applied or apply them manually.

### DNS

All `*.vsedomatut.com` subdomains route through Cloudflare Tunnel to the laptop.
**Never create A records in Cloudflare** — the wildcard CNAME handles everything.

### Important Notes

- This app was migrated from Hetzner to laptop on 2026-03-24
- The laptop server is a SEPARATE machine from the dev machine (C:\dev)
- `docker.exe` on the dev machine shows LOCAL containers, not server containers
- Use SSH or Coolify API to interact with production containers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filippmiller) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
