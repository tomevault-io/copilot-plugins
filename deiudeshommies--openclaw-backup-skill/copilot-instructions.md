## openclaw-backup-skill

> Universal OpenClaw configuration backup and restore tool with automatic detection. Intelligently discovers and backs up all OpenClaw components regardless of installation location, agent configuration, or enabled integrations. Works across different OpenClaw versions, platforms (macOS/Linux/Windows), and deployment types. Use when backing up any OpenClaw instance, migrating between machines, creating disaster recovery snapshots, or preserving configuration state. Trigger on "backup", "backup openclaw", "backup config", "export settings", "save configuration", "backup my setup", "create backup", "disaster recovery", "backup instance".


# Universal OpenClaw Configuration Backup

## Overview
OpenClaw is highly customizable, which means every instance is unique. This skill automatically detects your specific configuration and creates a complete backup without requiring manual configuration. Whether you have 3 agents or 30, use Telegram or Slack, run on macOS or Linux, this skill adapts to your setup.

**Key Features:**
- 🔍 **Auto-detection** - Discovers all components automatically
- 🌍 **Cross-platform** - Works on macOS, Linux, Windows
- 🎯 **Version-agnostic** - Compatible with all OpenClaw versions
- 🔒 **Safe** - Never assumes files exist, handles missing components gracefully
- 📦 **Complete** - Backs up everything that matters

---

## How It Works: Discovery-Based Backup

Unlike traditional backup tools that assume a fixed structure, this skill:

1. **Discovers** your OpenClaw installation location
2. **Scans** for all configuration files and directories
3. **Detects** enabled integrations and agents
4. **Backs up** only what exists in your instance
5. **Documents** what was backed up in a manifest

**No hardcoded paths. No assumptions. Just intelligent detection.**

---

## What Gets Backed Up (Auto-Detected)

### Core Configuration (if present)
- **openclaw.json** - Main configuration file
- **.env** - Environment variables and API keys
- **BOOT.md** - Startup configuration (if exists)
- **RECOVERY.md** - Recovery instructions (if exists)
- **config.yaml** / **config.toml** - Alternative config formats
- All backup files matching patterns: `*.bak`, `*.backup-*`, `*.old`

### Agent Configurations (all discovered agents)
The skill automatically scans your agents directory and backs up:
- All agent subdirectories (regardless of names)
- Agent-specific configuration files
- Agent prompts and instructions
- Model caches (if present)
- Session histories (optional, can be excluded)

**Example:** If you have agents named `coder`, `writer`, `analyst`, all will be backed up automatically.

### Authentication & Credentials (if configured)
Detects and backs up credentials for:
- Messaging platforms (Telegram, Slack, Discord, Feishu, etc.)
- OAuth tokens and API keys
- Bot configurations and pairing info
- Channel authorization settings
- Any files in `credentials/` or `auth/` directories

### Data & Knowledge (if present)
- **cron/** or **scheduler/** - Scheduled jobs and execution history
- **shared-learnings/** or **knowledge/** - Knowledge base
- **memory/** - Persistent memory and knowledge graphs
- **subagents/** - Subagent execution records
- **workspace/** - Workspace configurations and state
- **logs/** - Recent logs (optional)

### External Integrations (auto-detected)
If you have Claude, Cursor, or other IDE integrations:
- Searches common config locations (`~/.claude/`, `~/.cursor/`, etc.)
- Backs up integration configs if found
- Skips if not present

---

## Step 1: Quick Backup (Default Mode)

The fastest way to create a complete backup.

**Command:**
```
"backup my openclaw config"
"create a backup of my setup"
"backup everything"
```

**What happens:**
1. **Detects** OpenClaw installation directory
2. **Scans** for all configuration files
3. **Discovers** all agents (regardless of names)
4. **Identifies** enabled integrations
5. **Creates** timestamped backup directory
6. **Copies** all discovered components
7. **Generates** backup manifest and restore guide
8. **Verifies** backup integrity

**Output structure (example):**
```
<backup-location>/openclaw-backup-YYYYMMDD-HHMMSS/
├── openclaw-config/          # Core configs (detected)
│   ├── openclaw.json
│   ├── .env
│   ├── BOOT.md              # (if exists)
│   └── *.bak                # (if exists)
├── openclaw-data/            # All data (detected)
│   ├── agents/              # All discovered agents
│   │   ├── <agent-1>/
│   │   ├── <agent-2>/
│   │   └── ...
│   ├── cron/                # (if exists)
│   ├── credentials/         # (if exists)
│   ├── shared-learnings/    # (if exists)
│   ├── memory/              # (if exists)
│   └── workspace/           # (if exists)
├── integrations/             # External integrations (if detected)
│   └── claude/              # (if ~/.claude/ exists)
├── backup-manifest.json      # What was backed up
└── README.md                 # Restore guide
```

**Backup location priority:**
1. User-specified location (if provided)
2. Desktop directory (if exists and writable)
3. User's home directory under `~/openclaw-backups/`
4. Current directory as fallback

**Time estimate:** 30-90 seconds depending on instance size

---

## Step 2: Selective Backup

Backup only specific components.

**Available components (auto-detected):**

| Component | Command Trigger | What's Included |
|---|---|---|
| **Config only** | "backup just config files" | Core configuration files only |
| **Agents only** | "backup agent configurations" | All discovered agent directories |
| **Credentials** | "backup my credentials" | All auth tokens and keys |
| **Knowledge** | "backup shared learnings" | Knowledge base and memory |
| **Scheduled tasks** | "backup scheduled tasks" | Cron/scheduler configurations |
| **Integrations** | "backup integrations" | External tool configs |

**Example:**
```
User: "I just want to backup my agent configurations"

System:
1. Scans agents directory
2. Finds: agent-a/, agent-b/, agent-c/
3. Creates backup with only these agents
4. Reports: "Backed up 3 agents: agent-a, agent-b, agent-c"
```

---

## Step 3: Scheduled Backups

Set up automatic backups using your system's scheduler.

**Command:**
```
"setup daily backups"
"create automatic backup schedule"
```

**What happens:**
1. **Detects** available scheduler (cron, launchd, systemd timer, Task Scheduler)
2. **Configures** backup job using detected scheduler
3. **Sets** retention policy (configurable, default: keep last 7)
4. **Enables** notifications (if messaging integration detected)
5. **Tests** backup job

**Scheduler detection:**
- **macOS**: Uses launchd or cron
- **Linux**: Uses systemd timer or cron
- **Windows**: Uses Task Scheduler
- **Docker**: Uses container scheduler or cron

**Retention policy (configurable):**
- Keep last N backups (default: 7)
- Delete backups older than X days (default: 30)
- Minimum backups to keep (default: 3)

**Notifications (if available):**
- Sends completion status to configured messaging platform
- Falls back to log file if no messaging integration

---

## Step 4: Restore from Backup

Recover your configuration from any backup.

**Command:**
```
"restore from backup"
"recover my openclaw config"
```

**Restore process:**
1. **Locates** backup directory
2. **Reads** backup manifest
3. **Detects** current OpenClaw installation
4. **Validates** compatibility
5. **Stops** OpenClaw services (if running)
6. **Restores** components from manifest
7. **Verifies** restored files
8. **Restarts** services

**Safety features:**
- Creates backup of current config before restore
- Asks for confirmation before overwriting
- Validates backup integrity first
- Supports partial restore (only specific components)
- Rollback option if restore fails

**Partial restore:**
```
"restore just agents from backup"
"restore credentials from backup"
"restore config files from backup"
```

---

## Step 5: Cross-Platform Migration

Moving between different systems? This skill handles it.

**Supported migrations:**
- macOS → macOS
- macOS → Linux
- Linux → Linux
- Linux → macOS
- Windows → Windows
- Windows → Linux (with path translation)
- Docker → Native
- Native → Docker

**Migration workflow:**

**On source machine:**
```
"create migration backup"
```
This creates a platform-independent backup with:
- Relative paths instead of absolute
- Platform-agnostic configuration
- Compatibility metadata

**On target machine:**
```
"restore migration backup from <location>"
```
The skill automatically:
- Translates paths for target platform
- Adjusts file permissions
- Updates platform-specific configs
- Validates compatibility

**Post-migration verification:**
```
"verify openclaw installation"
```
Checks:
- All agents are accessible
- Configuration is valid
- Services can start
- Integrations are connected

---

## Advanced Features

### 1. Incremental Backup

Only backup changed files since last backup.

```
"create incremental backup"
```

**Benefits:**
- Faster backup (only changed files)
- Smaller backup size
- Tracks changes over time

### 2. Encrypted Backup

Encrypt sensitive data before backup.

```
"create encrypted backup"
```

**What gets encrypted:**
- .env file (API keys)
- credentials/ directory
- Any files matching `*secret*`, `*token*`, `*key*`

**Encryption method:**
- Uses GPG if available
- Falls back to platform encryption (Keychain, Secret Service)
- Prompts for password

### 3. Compressed Backup

Create a single compressed archive.

```
"create compressed backup"
```

**Output:** `openclaw-backup-YYYYMMDD-HHMMSS.tar.gz`

**Compression:**
- Uses tar + gzip on Unix
- Uses zip on Windows
- Typically reduces size by 60-80%

### 4. Remote Backup

Backup to remote location.

```
"backup to <remote-path>"
"backup to /mnt/nas/backups"
"backup to \\server\backups"
```

**Supported:**
- Network drives (SMB, NFS)
- Cloud mounts (rclone, s3fs)
- SSH/SFTP locations
- Any writable path

### 5. Backup Verification

Verify backup integrity without restoring.

```
"verify backup integrity"
"check backup <backup-name>"
```

**Checks:**
- All files in manifest exist
- File sizes match
- Checksums are valid (if available)
- Backup is restorable

### 6. Backup Comparison

Compare two backups to see what changed.

```
"compare backups"
"diff backup-1 and backup-2"
```

**Shows:**
- Added files
- Removed files
- Modified files
- Configuration changes

---

## Configuration Options

### Backup Location

Set default backup location:

```
"set backup location to ~/Documents/openclaw-backups"
```

Or use environment variable:
```bash
export OPENCLAW_BACKUP_DIR=~/Documents/openclaw-backups
```

### Exclusions

Exclude specific files or directories:

```
"exclude session histories from backup"
"exclude logs from backup"
```

Or create `.backupignore` file:
```
# Exclude patterns
*.log
sessions/
cache/
*.tmp
```

### Retention Policy

Configure how long to keep backups:

```
"keep last 14 backups"
"delete backups older than 60 days"
```

---

## Troubleshooting

### Issue: "Cannot detect OpenClaw installation"

**Cause:** OpenClaw not in standard location

**Solution:**
```
"backup openclaw from /custom/path/to/openclaw"
```

Or set environment variable:
```bash
export OPENCLAW_HOME=/custom/path/to/openclaw
```

---

### Issue: "Backup location not writable"

**Cause:** No permission to write to default location

**Solution:**
```
"backup to ~/Documents"
```

Or fix permissions:
```bash
chmod 755 ~/Desktop
```

---

### Issue: "Some components not backed up"

**Cause:** Components don't exist or aren't accessible

**Solution:**
Check backup manifest to see what was skipped:
```
cat backup-manifest.json
```

This is normal - the skill only backs up what exists.

---

### Issue: "Restore fails with version mismatch"

**Cause:** Backup from different OpenClaw version

**Solution:**
The skill will attempt automatic migration. If it fails:
```
"restore with compatibility mode"
```

Or upgrade/downgrade OpenClaw to match backup version.

---

### Issue: "Backup is too large"

**Cause:** Including session histories or large caches

**Solution:**
```
"backup without session histories"
"backup without caches"
```

Or use incremental backup:
```
"create incremental backup"
```

---

## Platform-Specific Notes

### macOS
- Default location: `~/Desktop/openclaw-backups/`
- Uses launchd for scheduled backups
- Supports Keychain for encryption

### Linux
- Default location: `~/openclaw-backups/`
- Uses systemd timer or cron for scheduled backups
- Supports Secret Service for encryption

### Windows
- Default location: `%USERPROFILE%\Desktop\openclaw-backups\`
- Uses Task Scheduler for scheduled backups
- Supports DPAPI for encryption

### Docker
- Detects container environment
- Backs up volumes and bind mounts
- Handles container-specific paths

---

## Best Practices

### 1. Regular Backups

**Recommended schedule:**
- **Daily use**: Daily automated backups
- **Weekly use**: Weekly automated backups
- **Occasional use**: Manual backup before changes

**Setup:**
```
"setup daily backups at 2am"
```

---

### 2. Before Major Changes

**Always backup before:**
- Upgrading OpenClaw
- Modifying core configuration
- Installing new skills or plugins
- Changing agent configurations
- Updating API keys

**Quick backup:**
```
"quick backup before changes"
```

---

### 3. Test Restores

**Monthly verification:**
```
"verify latest backup"
```

**Quarterly test restore:**
1. Create test backup
2. Restore to temporary location
3. Verify all components work
4. Document restore time

---

### 4. Multiple Backup Locations

**Recommended strategy:**
- **Local**: Fast access, quick restore
- **Network**: Shared access, redundancy
- **Cloud**: Off-site, disaster recovery

**Setup:**
```
"backup to ~/backups"              # Local
"backup to /mnt/nas/backups"       # Network
"backup to ~/cloud/backups"        # Cloud (via sync)
```

---

### 5. Secure Sensitive Data

**For backups containing credentials:**
```
"create encrypted backup"
```

**For cloud storage:**
```bash
# Encrypt before upload
gpg -c openclaw-backup-*.tar.gz
# Upload .gpg file only
```

**Never:**
- Upload unencrypted backups to public cloud
- Share backups without removing credentials
- Commit backups to version control

---

## Quick Reference

### Common Commands

```
"backup my openclaw config"           → Full backup
"backup just config files"            → Config only
"backup just agents"                  → Agents only
"create encrypted backup"             → Encrypted full backup
"create compressed backup"            → Compressed archive
"backup to ~/Documents"               → Custom location
"setup daily backups"                 → Automated backups
"list all backups"                    → Show backups
"verify backup integrity"             → Check backup
"restore from backup"                 → Full restore
"restore just agents"                 → Partial restore
```

### File Locations (Auto-Detected)

- **OpenClaw config**: Detected from installation
- **Backup location**: User-specified > Desktop > ~/openclaw-backups/
- **Manifest**: `<backup-dir>/backup-manifest.json`
- **Restore guide**: `<backup-dir>/README.md`

### Backup Size Estimates

- **Config only**: ~1-5 MB
- **Config + Agents**: ~10-100 MB
- **Full backup**: ~50-500 MB
- **With histories**: ~500 MB - 2 GB

*Actual size depends on your instance configuration*

---

## Success Criteria

You'll know this skill is working when:
- ✅ Backup completes without errors
- ✅ Manifest lists all your components
- ✅ Backup can be restored successfully
- ✅ No hardcoded paths in backup
- ✅ Works on different machines
- ✅ Handles missing components gracefully

---

## When to Use This Skill

**✅ Use this skill when:**
- Backing up any OpenClaw instance
- Migrating between machines
- Before major configuration changes
- Setting up disaster recovery
- Sharing configuration (after sanitizing credentials)
- Testing configuration changes safely

**❌ Don't use for:**
- Backing up project code (use git)
- Backing up large datasets (use dedicated backup tools)
- Real-time synchronization (use sync tools)
- Version control of code (use git)

---

**This skill adapts to YOUR OpenClaw instance, not the other way around.** 🎯

---
> Source: [DeiuDesHommies/openclaw-backup-skill](https://github.com/DeiuDesHommies/openclaw-backup-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
