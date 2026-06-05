## callcenter-issabel5

> This file provides guidance to Gemini CLI when working with code in this repository.

# GEMINI.md

This file provides guidance to Gemini CLI when working with code in this repository.

## Project Overview

Issabel Call Center module (v4.0.0.6) for Issabel 5 - a predictive dialer and call center solution built on Asterisk PBX. Runs on Rocky 8 with PHP 7.x, Asterisk 16/18, and MySQL/MariaDB.

## Installation Commands

```bash
# Full installation (run as root)
cd /usr/src
git clone https://github.com/ISSABELPBX/callcenter-issabel5.git callcenter
cd callcenter
bash build/5.0/install-issabel-callcenter.sh

# Service management
systemctl start issabeldialer
systemctl status issabeldialer
systemctl stop issabeldialer

# Manual dialer start (debug mode, runs in foreground)
su - asterisk -c "/opt/issabel/dialer/dialerd -d"
```

## Architecture

### Multi-Process Dialer Daemon

The core is a PHP daemon (`/opt/issabel/dialer/dialerd`) that spawns child processes:

```
HubProcess (master coordinator)
├── AMIEventProcess     - Asterisk Manager Interface events (queue/agent status)
├── CampaignProcess     - Outgoing campaign orchestration, call scheduling
├── SQLWorkerProcess    - Database persistence, call progress logging
├── ECCPProcess         - ECCP protocol server for agent consoles
└── ECCPWorkerProcess   - Individual agent connection handlers
```

Processes communicate via `TuberiaMensaje` (message pipe) class.

### Web Modules Structure

28 modules in `/var/www/html/modules/` (copied from `modules/`), each following:

```
module_name/
├── index.php              # Entry point with _moduleContent() function
├── configs/default.conf.php  # DSN and module config
├── libs/*.class.php       # Business logic (PaloSanto* classes)
├── themes/default/*.tpl   # Smarty templates
└── lang/                  # i18n translations
```

### Key Module Categories

- **Agent Console**: `agent_console` - main agent interface with ECCP client
- **Campaigns**: `campaign_out` (outgoing), `campaign_in` (incoming)
- **Reports**: `calls_detail`, `calls_per_agent`, `calls_per_hour`, `hold_time`, etc.
- **Admin**: `agents`, `queues`, `break_administrator`, `form_designer`

### Database

Schema in `setup/call_center.sql`. Key tables:
- `agent` - Agent definitions (SIP/IAX2/PJSIP types)
- `calls` / `call_entry` - Outgoing/incoming call records
- `campaign` / `campaign_entry` - Campaign configurations
- `audit` - Agent session tracking (login/pause/logout)
- `form`, `form_field`, `form_data_recolected` - Custom form system

### ECCP Protocol

Custom binary protocol for agent console ↔ dialer communication. Spec in `setup/dialer_process/dialer/Protocolo ECCP.txt`. Implementation in `ECCPConn.class.php`.

## Key Files

- `setup/installer.php` - Database schema creation and migrations
- `setup/dialer_process/dialer/dialerd` - Main daemon entry point
- `setup/dialer_process/dialer/AMIEventProcess.class.php` - Asterisk event handler (3,847 lines)
- `setup/dialer_process/dialer/ECCPConn.class.php` - ECCP protocol (3,300 lines)
- `modules/agent_console/index.php` - Agent console (85KB)
- `menu.xml` - Issabel menu integration

## Development Notes

- Testing and development is done on the live system, don't modify the repo before I ask you
- Dialer runs as `asterisk` user (refuses to run as root)
- Dialer logs: `/opt/issabel/dialer/dialerd.log`
- WebServer logs: `/var/log/httpd/`
- Asterisk logs: `/var/log/asterisk/`
- PID file: `/opt/issabel/dialer/dialerd.pid`
- Uses PDO for database access in dialer, mysqli in web modules
- Smarty templates use `_tr()` function for i18n
- Agent types: `Agent`, `SIP`, `IAX2`, `PJSIP`
- MySQL root password can be found in /etc/issabel.conf (mysqlrootpwd)
- Asterisk conf files: `/etc/asterisk/`
- Local git repos exist in both `/opt/issabel/dialer/` and `/var/www/html/modules/` for changes tracking
- **IMPORTANT**: Always use `/bin/cp` instead of `cp` for file operations to avoid shell alias issues
- Don't assume, always use debuggers if required to check for a value

---

## Change History

See [CHANGES.md](CHANGES.md) for detailed bug fixes and modifications.

---

## Technical Notes

### Local Extension Calls
- Multiple Local channels with different uniqueids are created
- `actualchannel` stores the remote party's real channel (e.g., SIP/103-xxx)
- Call search order in msg_Hangup: uniqueid -> uniqueidlink -> actualchannel

### Agent Types (app_agent_pool)
- **Agent type**: Uses `Local/XXXX@agents` with `StateInterface=Agent:XXXX`
- **Callback type**: Uses direct SIP/PJSIP extension
- app_agent_pool replaces deprecated chan_agent (Asterisk 12+)

### Monitoring Modules
- Uses AJAX long polling with Ember.js (NOT full page refresh)
- Agent status updates in real-time without page reload with SSE

### Testing
- Outgoing campaign calls goes through the dialplan by default but can be explicitly set to use specific trunk
- External calls uses SIP/120Issabel4 trunk

---
> Source: [ISSABELPBX/callcenter-issabel5](https://github.com/ISSABELPBX/callcenter-issabel5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-04 -->
