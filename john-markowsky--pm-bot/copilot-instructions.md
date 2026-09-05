## production-vm-workflow

> Git-based dev→prod workflow and production VM management patterns


# Production VM Workflow

## Core Principle

**⚠️ CRITICAL: NEVER edit production VM files directly via SSH for configuration changes.**

Always use git workflow: edit locally → commit to dev → deploy via `deploy-prod.sh`.

## Git-Based Workflow (REQUIRED)

### Standard Process

1. **Edit locally** (dev branch)
   ```bash
   git checkout dev
   # Edit files (docker-compose.v2.yml, .env, code, etc.)
   ```

2. **Test locally**
   ```bash
   docker compose -f v2/docker-compose.v2.yml up -d
   # Test changes
   ```

3. **Commit to dev**
   ```bash
   git add .
   git commit -m "feat: description of changes"
   git push origin dev
   ```

4. **Deploy to prod**
   ```bash
   cd /home/jgm/Git/nexus
   ./scripts/deploy-prod.sh
   ```

5. **Verify on prod**
   - Check services: `sshpass -p 'K0smo45!!!' ssh administrator@192.168.0.102 "cd /opt/nexus/v2 && docker compose ps"`
   - Test functionality on `prod.nexus-trade.top`

### What Goes Through Git Workflow

✅ **Configuration files**: `docker-compose.v2.yml`, `.env`, service configs  
✅ **Code changes**: All application code  
✅ **Service updates**: New services, version updates  
✅ **Infrastructure changes**: Network configs, volumes, etc.  
✅ **Documentation updates**: README, docs (committed WITH code)

## When SSH is Acceptable (Limited)

### Read-Only Operations ✅

**Always allowed for debugging and monitoring:**

```bash
# Check logs (use Loki MCP when possible)
docker logs <service>

# Health checks
docker compose ps
docker stats

# Status verification
docker inspect <container>
docker exec <container> env

# Read-only file inspection
cat /opt/nexus/v2/docker-compose.v2.yml
ls -la /opt/nexus/v2/
```

### Emergency Hotfixes ⚠️

**ONLY for critical production issues that cannot wait for git workflow**

**Process**:
1. SSH edit on prod VM (document reason)
2. **IMMEDIATELY commit to prod branch**:
   ```bash
   sshpass -p 'K0smo45!!!' ssh administrator@192.168.0.102
   cd /opt/nexus/v2
   # Make emergency fix
   git add .
   git commit -m "EMERGENCY: [reason] - [what was fixed]

   Example: EMERGENCY: Production down - removed broken env var causing crash"
   git push origin prod
   ```
3. **Follow up**: Merge fix to dev branch within 24 hours
4. **Document**: Add to incident log or GitHub issue

### One-Time Data Fixes ⚠️

- Database migrations (via db_service, not direct SQL)
- Data corrections (must be scripted and committed)
- Configuration hotfixes (commit immediately after)

## Deployment Script

**File**: `scripts/deploy-prod.sh`

**What it does**:
- Verifies you're on `dev` branch
- Checks for clean working tree
- Merges `dev` → `prod` branch
- Syncs backups to VM
- Deploys latest `prod` branch to VM
- Runs health checks

**Usage**:
```bash
cd /home/jgm/Git/nexus
./scripts/deploy-prod.sh
```

**Prerequisites**:
- Must be on `dev` branch
- Working tree clean (no uncommitted changes)
- SSH access to VM configured

## VM Structure

### Production VM
- **Host**: `192.168.0.102`
- **User**: `administrator`
- **App Directory**: `/opt/nexus/v2` (git repo on `prod` branch)
- **SSH**: `sshpass -p 'K0smo45!!!' ssh administrator@192.168.0.102`
- **Domain**: `prod.nexus-trade.top`

### Development Host
- **Host**: `192.168.0.103` (dev VM)
- **App Directory**: `/opt/nexus/v2`
- **Domain**: `dev.nexus-trade.top`

## Rollback Procedures

### Quick Rollback (On VM)

```bash
sshpass -p 'K0smo45!!!' ssh administrator@192.168.0.102
cd /opt/nexus/v2
git log --oneline -5  # Find previous good commit
git checkout <commit-hash>
docker compose -f docker-compose.v2.yml -f ../docker-compose.prod.yml up -d --build
```

### Full Rollback (Git-Based)

```bash
# Local
git checkout prod
git reset --hard HEAD~1  # Or specific commit
git push origin prod --force

# Then redeploy
./scripts/deploy-prod.sh
```

## Common Mistakes

### ❌ BAD: Direct SSH Edit

```bash
sshpass -p 'K0smo45!!!' ssh administrator@192.168.0.102
vi /opt/nexus/v2/docker-compose.v2.yml  # ❌ DON'T DO THIS
docker compose up -d  # ❌ Changes not in git
```

**Problems**:
- No version control
- No audit trail
- Can't rollback easily
- Changes lost if VM rebuilt

### ✅ GOOD: Git Workflow

```bash
# Local edit
vi v2/docker-compose.v2.yml
git commit -am "feat: update Grafana config"
./scripts/deploy-prod.sh
```

**Benefits**:
- Version controlled
- Audit trail
- Easy rollback
- Changes persist

## Environment Identification

### From User Context

**Domain-based**:
- `prod.nexus-trade.top` → **PROD** (192.168.0.102)
- `dev.nexus-trade.top` → **DEV** (192.168.0.103)

**Branch-based**:
- `prod` branch → **PROD**
- `dev` branch → **DEV**

**From Issue Reports**:
- "Production is down" → Query prod logs
- "Dev environment error" → Query dev logs
- Check domain in error message/URL

### Log Querying Context

**Query DEV logs when**:
- Working on `dev` branch
- Testing locally
- User reports issue on `dev.nexus-trade.top`
- Debugging development features

**Query PROD logs when**:
- User reports production issue
- Issue on `prod.nexus-trade.top`
- Production monitoring/alerts
- Post-deployment verification

See [MCP Cheat Sheet](mdc:.cursor/rules/mcp-cheat-sheet.mdc) for Loki query patterns.

## Best Practices

1. **Always test locally first** before deploying
2. **Use feature branches** for complex changes
3. **Keep commits atomic and descriptive**
4. **Monitor services after deployment**
5. **Document emergency fixes immediately**
6. **Regular cleanup of old branches**
7. **Use Loki MCP for log queries** (not docker logs)

## References

- [Dev/Prod Workflow](mdc:DEV-PROD-WORKFLOW.md) - Complete workflow documentation
- [Deployment Script](mdc:scripts/deploy-prod.sh) - Automated deployment
- [MCP Cheat Sheet](mdc:.cursor/rules/mcp-cheat-sheet.mdc) - Log querying patterns
- [Core Workflow](mdc:.cursor/rules/core-workflow.mdc) - General development patterns

---
> Source: [john-markowsky/PM-Bot](https://github.com/john-markowsky/PM-Bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
