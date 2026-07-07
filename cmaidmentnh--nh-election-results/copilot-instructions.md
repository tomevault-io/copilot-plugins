## nh-election-results

> **DO NOT CHANGE THESE PORTS. EVER.**

# NH Election Results - Claude Reference

## CRITICAL: Server Port Assignments

**DO NOT CHANGE THESE PORTS. EVER.**

| App | Port | Service | Domains |
|-----|------|---------|---------|
| nh-civic-crm | 5000 | nh-civic-crm.service | actioncenter.winthehouse.gop, actioncenter.rebuildnh.com, actioncenter.nhfc-ontarget.org |
| nh-legislators-api | 5001 | nh-legislators-api.service | (internal API) |
| secure-messenger | 5003 | secure-messenger.service | (internal) |
| nh-whip-count | 5004 | nh-whip-count.service | whip.nhhouse.gop |
| nh-legislators-api (alt?) | 5005 | (check) | (internal) |
| nh-election-results | 5006 | nh-election-results.service | elections.nhhouse.gop |
| granitesend | 5007 | granitesend.service | granitesend.com |

All apps run on server: `138.197.20.97`

---

## Deployment

```bash
# Deploy to production (elections.nhhouse.gop)
ssh root@138.197.20.97 "cd /opt/nh-election-results && git pull && systemctl restart nh-election-results"
```

## Server Access

```bash
# SSH to production server
ssh root@138.197.20.97

# App directory on server
cd /opt/nh-election-results
```

## Checking Logs

```bash
ssh root@138.197.20.97 "journalctl -u nh-election-results -n 50 --no-pager"
```

## Database

Local SQLite database: `nh_elections.db`

Key tables:
- `results` - Vote counts by municipality/candidate
- `races` - Race metadata (district, county, seats)
- `candidates` - Candidate info
- `elections` - Election year/type
- `offices` - Office names
- `district_compositions` - Which towns are in which districts
- `voter_registration` - Ballots cast data for turnout
- `users` - Admin/user accounts for results entry
- `result_audit` - Audit log for result changes

## Git Workflow

Always commit, push, AND deploy:
```bash
git add -A && git commit -m "message" && git push
ssh root@138.197.20.97 "cd /opt/nh-election-results && git pull && systemctl restart nh-election-results"
```

## Admin Portal

- Login: https://elections.nhhouse.gop/login
- Admin panel: /admin/
- Results entry: /entry/

---
> Source: [cmaidmentnh/nh-election-results](https://github.com/cmaidmentnh/nh-election-results) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
