## pigenny

> Automates deployment of gen_server.py updates on the Olimex.

# PiGenny Generator Controller - Project Context

## Overview
PiGenny is a hybrid Raspberry Pi + Olimex iMX233 system that automatically starts a backup generator when battery state of charge drops below a threshold. This replaces the older single-board Arch Linux ARM system.

## Architecture

### Hardware Components
- **Raspberry Pi** (momspi.local / 10.2.242.1)
  - RS-485 communication with LuxPower/GSL inverter (19200 baud on /dev/ttySC1)
  - TCP client to Olimex generator controller
  - Runs control logic and monitoring (monitor.py)

- **Olimex iMX233-OLinuXino-MAXI** (10.2.242.109)
  - I2C control of MOD-IO relay board (address 0x58)
  - TCP server for generator commands (gen_server.py on port 9999)
  - Direct relay control for generator start/stop/charger

- **LuxPower/GSL Inverter**
  - RS-485 Modbus RTU (slave address 1)
  - Reports battery SOC, voltage, PV power, charge/discharge

- **Generator**
  - Controlled via 4 relays: IGN (ignition), START (starter motor), GLOW (glow plugs), CHARGER (AC transfer)
  - Status feedback via MOD-IO input register 0x20

### Network
- Direct-wire Ethernet between Pi and Olimex
- Pi: 10.2.242.1
- Olimex: 10.2.242.109
- Isolated network (no external connectivity)

## CRITICAL: Understanding Battery Charging

### Inverter Register Interpretation

**IMPORTANT:** The LuxPower inverter has separate registers for different power flows:

- **Register 10 (charge_power_w)**: PV SOLAR charging power ONLY
  - This shows how much power is coming from solar panels to charge the battery
  - This will be 0W at night or when generator is the charging source

- **Register 11 (discharge_power_w)**: Battery discharge to loads
  - Power being drawn from battery to supply house loads

- **Register 9 (pv_power_w)**: Total solar panel output

### How to Tell if Generator Charging is Working

**DO NOT rely on charge_power_w register!** It only tracks solar charging.

When the generator is running and charging the batteries, you should see:
1. **SOC (State of Charge) increasing** - e.g., 21% → 28% over time
2. **Battery voltage increasing** - e.g., 51.5V → 53.4V
3. **Discharge power lower than house load** - some load being supplied by generator

**If you see:** SOC rising, voltage rising, but charge_power_w = 0
- **This is NORMAL** - generator is charging, solar is not
- The inverter likely doesn't report AC/grid charging power in the Modbus registers we're reading

**If you see:** SOC falling, voltage falling, charge_power_w = 0
- **This is a problem** - no charging is happening
- Check relay states, AC wiring, generator running status

## Generator Control Sequence

### Start Sequence (gen_server.py)
1. Phase 1: Reset (0b0000) - 1 second
2. Phase 2: IGN on (0b1000) - 0.5 seconds
3. Phase 3: IGN + START (0b1001) - 10 seconds (main crank)
4. Phase 4: IGN + GLOW + START (0b1101) - 2 seconds (glow-assisted crank)
5. Phase 5: IGN only (0b1000) - 4 seconds (coast and verify)
6. Check status register: if running (status == 3):
   - 60 second warmup at idle
   - 20 second AC stabilization
   - Enable charger relay (0b1010 = IGN + CHARGER)
7. If not running: emergency shutdown (0b0000)

### Stop Sequence (gen_server.py)
1. Disconnect charger (0b1000 = IGN only)
2. Run at idle for 3 minutes cooldown
3. Full shutdown (0b0000)

**Note:** 3-minute cooldown is only needed if generator was under load. If generator was running but not actually charging (e.g., relay on but AC not connected), immediate shutdown is fine.

## Known Issues and Fixes

### Thread Leak Bug (Fixed 2026-01-02)

**Problem:** gen_server.py had a thread leak where each TCP connection created a new thread that was never cleaned up. After ~5 hours of hourly status checks, the system would exhaust thread limits and crash with "can't start new thread" error.

**Symptoms:**
- Pi logs show "Broken pipe" errors when trying to connect to Olimex
- gen_server.log shows "Server error: can't start new thread"
- Emergency relay shutdown triggered
- System in ERROR state, generator won't start even when SOC is low

**Fix Applied:**
- Added thread tracking and cleanup in gen_server.py
- Implemented MAX_CONCURRENT_CONNECTIONS limit (5)
- Active threads are cleaned up after each connection completes
- Added logging for thread cleanup operations

**File:** gen_server.py lines 51-61, 285-293, 353-382

**Deployment Note:** Restarting gen_server.py immediately shuts down the generator (relays set to 0b0000 on startup). Only deploy updates when generator is not needed or after a proper stop sequence.

## Maintenance Tools

### update_genserver.py (Olimex)
Automates deployment of gen_server.py updates on the Olimex.

**Usage:**
```bash
# On Olimex (run as root or with sudo)
sudo python2 /usr/local/bin/update_genserver.py [--source /path/to/gen_server.py]
```

**What it does:**
1. Verifies source file exists
2. Copies to /usr/local/bin/gen_server.py
3. Kills old process (SIGTERM, then SIGKILL if needed)
4. Waits for rc.local to auto-restart
5. Verifies new process started and is responding

**Default source:** `/home/derekja/gen_server.py`

**Remote deployment from Pi:**
```bash
sshpass -p 'Login123' ssh derekja@10.2.242.109 'echo Login123 | sudo -S python2 /usr/local/bin/update_genserver.py'
```

### genserverstatus.py (Olimex or Pi)
Query gen_server.py status and diagnostics.

**Usage:**
```bash
# On Olimex (local)
python2 /usr/local/bin/genserverstatus.py

# From Pi (remote)
python2 /home/derekja/pigenny/genserverstatus.py --host 10.2.242.109

# Compact format (single line)
python2 genserverstatus.py --format compact

# Key=value format (for scripting)
python2 genserverstatus.py --format kv
```

**Output includes:**
- Generator running state
- Relay states (IGN, START, GLOW, CHARGER)
- Input states (engine running sensor)
- Active thread count (thread leak monitoring)
- System uptime
- Memory usage percentage
- Disk space available

**Formats:**
- `human`: Multi-line human-readable (default)
- `compact`: Single-line summary for dashboards
- `kv`: Key=value pairs for parsing/scripting

### Automated Health Monitoring

The Pi monitor.py automatically checks Olimex health metrics and logs them:

**Health check interval:** 3600 seconds (1 hour, configurable with `olimex_health_check_interval`)

**Metrics logged to Pi system logs:**
- Active threads (thread leak detection)
- Uptime (detect unexpected reboots)
- Memory usage (detect memory leaks)
- Disk space (prevent log/file exhaustion)

**View health logs:**
```bash
journalctl -u pigenny | grep "Olimex health"
```

**Example output:**
```
2026-01-02 15:30:00 | INFO | Olimex health: threads=2 uptime=6h45m memory=42% disk=1.2G free (35% used)
```

## Control Thresholds

### Default Settings (monitor.py)
- **Start threshold:** 40% SOC
- **Stop threshold:** 80% SOC
- **Poll interval:** 30 seconds
- **Log interval:** 600 seconds (10 minutes)
- **Olimex health check interval:** 3600 seconds (1 hour)
- **Generator max runtime:** 14400 seconds (4 hours)
- **Consecutive start failures before ERROR:** 3

### Systemd Service
- Service: pigenny.service
- Auto-restart: enabled (30 second delay)
- User: derekja
- Working directory: /home/derekja/pigenny

## File Locations

### Raspberry Pi
- Application: `/home/derekja/pigenny/monitor.py`
- Service: `/etc/systemd/system/pigenny.service`
- Tools: `/home/derekja/pigenny/genserverstatus.py`
- Logs: `/var/log/pigenny/data_YYYYMMDD.csv`
- System logs: `journalctl -u pigenny`

### Olimex iMX233
- Application: `/usr/local/bin/gen_server.py`
- Startup script: `/usr/local/bin/start_gen_server.sh`
- Maintenance tools:
  - `/usr/local/bin/update_genserver.py`
  - `/usr/local/bin/genserverstatus.py`
- Auto-start: `/etc/rc.local` (calls start_gen_server.sh)
- Logs: `/var/log/gen_server.log`
- User: root (required for I2C access)

## SSH Access

### Pi
- Hostname: momspi.local
- User: derekja
- Auth: SSH key (Codex has ~/.ssh/momspi)
- From local machine: `ssh -i ~/.ssh/momspi derekja@momspi.local`

### Olimex (via Pi)
- IP: 10.2.242.109
- User: derekja
- Password: Login123
- From Pi: `sshpass -p 'Login123' ssh derekja@10.2.242.109`
- From local: `ssh -i ~/.ssh/momspi derekja@momspi.local "sshpass -p 'Login123' ssh derekja@10.2.242.109 'command'"`

**Note:** Credentials documented here because Olimex is on isolated network with no external access. sshpass must be installed on Pi for automated access.

## Operational Notes

### Normal Operation
- monitor.py runs continuously via systemd
- Polls inverter every 30 seconds
- Logs to CSV every 10 minutes
- When SOC < 40%: starts generator automatically
- When SOC >= 80% or 4 hours elapsed: stops generator
- Generator runs with 60s warmup + 20s AC stabilization before charger enable
- Shutdown includes 3-minute idle cooldown (if under load)

### Error Recovery
- After 3 consecutive start failures: enters ERROR state
- In ERROR state: won't attempt more starts (prevents damage)
- Systemd auto-restart (every 30s) will retry after Pi reboot or manual service restart
- Manual recovery: `sudo systemctl restart pigenny`

### Manual Generator Control
From Pi:
```bash
cd pigenny
python3 -c "
import socket
s = socket.socket()
s.connect(('10.2.242.109', 9999))
s.recv(1024)
s.send(b'START\n')  # or STOP, STATUS, PING
data = s.recv(4096)
print(data.decode())
s.close()
"
```

### Checking Generator Status
```bash
ssh -i ~/.ssh/momspi derekja@momspi.local "sshpass -p 'Login123' ssh derekja@10.2.242.109 'tail -50 /var/log/gen_server.log'"
```

### Checking Pi Status
```bash
ssh -i ~/.ssh/momspi derekja@momspi.local "journalctl -u pigenny --since '1 hour ago' -n 50"
```

### Reading CSV Logs
```bash
ssh -i ~/.ssh/momspi derekja@momspi.local "tail -20 /var/log/pigenny/data_$(date +%Y%m%d).csv"
```

## Deployment Process

### Automated Deployment (Recommended)

Use `deploy.sh` for fully automated deployment to both Pi and Olimex:

```bash
cd pigenny
./deploy.sh
```

**Options:**
- `--skip-pi` - Skip Pi deployment
- `--skip-olimex` - Skip Olimex deployment
- `--force` - Deploy even if generator is running (DANGEROUS!)

**What it does:**
1. Checks generator is stopped (safety check)
2. Deploys monitor.py and genserverstatus.py to Pi
3. Restarts pigenny service on Pi
4. Deploys gen_server.py and tools to Olimex
5. Uses update_genserver.py to restart gen_server
6. Reboots Olimex if needed
7. Verifies both systems are operational

**The script handles all SSH, file copying, service restarts, and verification automatically.**

### Manual Deployment (if needed)

#### Deploying to Pi
```bash
scp -i ~/.ssh/momspi pigenny/monitor.py derekja@momspi.local:/home/derekja/pigenny/
ssh -i ~/.ssh/momspi derekja@momspi.local "sudo systemctl restart pigenny"
```

#### Deploying to Olimex
**WARNING:** Restarting gen_server.py immediately shuts down generator!

**Only deploy when:**
- Generator is not running
- OR after proper stop sequence (with cooldown if under load)
- OR willing to accept immediate shutdown

**Recommended Process (using update_genserver.py):**
```bash
# Copy gen_server.py to Pi
scp -i ~/.ssh/momspi pigenny/gen_server.py derekja@momspi.local:/tmp/

# Copy to Olimex home directory
ssh -i ~/.ssh/momspi derekja@momspi.local \
  "sshpass -p 'Login123' scp /tmp/gen_server.py derekja@10.2.242.109:/home/derekja/"

# Run automated update script (kills old process, copies, verifies restart)
ssh -i ~/.ssh/momspi derekja@momspi.local \
  "sshpass -p 'Login123' ssh derekja@10.2.242.109 'echo Login123 | sudo -S python2 /usr/local/bin/update_genserver.py'"
```

**Alternative: Reboot Olimex (if update script unavailable or fails)**
```bash
# Copy files as above, then:
ssh -i ~/.ssh/momspi derekja@momspi.local \
  "sshpass -p 'Login123' ssh derekja@10.2.242.109 'sudo reboot'"

# Wait ~30 seconds for reboot
# gen_server.py auto-starts via /etc/rc.local
```

**Deploying update_genserver.py itself:**
```bash
scp -i ~/.ssh/momspi pigenny/update_genserver.py derekja@momspi.local:/tmp/
ssh -i ~/.ssh/momspi derekja@momspi.local \
  "sshpass -p 'Login123' scp /tmp/update_genserver.py derekja@10.2.242.109:/home/derekja/ && \
   sshpass -p 'Login123' ssh derekja@10.2.242.109 'echo Login123 | sudo -S cp /home/derekja/update_genserver.py /usr/local/bin/ && sudo chmod +x /usr/local/bin/update_genserver.py'"
```

**Deploying genserverstatus.py:**
```bash
# To Olimex
scp -i ~/.ssh/momspi pigenny/genserverstatus.py derekja@momspi.local:/tmp/
ssh -i ~/.ssh/momspi derekja@momspi.local \
  "sshpass -p 'Login123' scp /tmp/genserverstatus.py derekja@10.2.242.109:/home/derekja/ && \
   sshpass -p 'Login123' ssh derekja@10.2.242.109 'echo Login123 | sudo -S cp /home/derekja/genserverstatus.py /usr/local/bin/ && sudo chmod +x /usr/local/bin/genserverstatus.py'"

# To Pi (for remote querying)
scp -i ~/.ssh/momspi pigenny/genserverstatus.py derekja@momspi.local:/home/derekja/pigenny/
ssh -i ~/.ssh/momspi derekja@momspi.local "chmod +x /home/derekja/pigenny/genserverstatus.py"
```

## Troubleshooting

### Generator won't start
1. Check Pi logs: `journalctl -u pigenny -n 100`
2. Check if in ERROR state (3+ failed starts)
3. Check Olimex connectivity: `ping 10.2.242.109` from Pi
4. Check gen_server.py running: `ps aux | grep gen_server` on Olimex
5. Check for thread leak: look for "can't start new thread" in /var/log/gen_server.log

### Charging not happening (actual problem)
1. Check SOC and voltage trends - if both rising, charging IS happening
2. Check relay status: STATUS command should show "IGN+CHARGER" when running
3. Check physical wiring and transfer switch
4. Verify generator is actually running (listen for engine)

### Monitor.py crashes/stops
1. Check systemd status: `systemctl status pigenny`
2. Check recent logs: `journalctl -u pigenny -n 100`
3. Systemd will auto-restart after 30 seconds
4. Manual restart: `sudo systemctl restart pigenny`

### CSV logs not being written
1. Check directory permissions: `ls -ld /var/log/pigenny`
2. Check disk space: `df -h`
3. Logs written every 10 minutes by default (configurable with --log-interval)

## Testing

### Test generator start (short run)
```bash
ssh -i ~/.ssh/momspi derekja@momspi.local \
  "cd pigenny && python3 monitor.py --soc-start 95 --soc-stop 97 --log-interval 30"
```
This will start immediately (current SOC < 95%) and stop quickly (target 97%)

### Test with current SOC
Check current level first:
```bash
ssh -i ~/.ssh/momspi derekja@momspi.local \
  "cd /usr/local/bin && python3 /home/derekja/luxpower_485/modbus_read_one.py \
   --port /dev/ttySC1 --baud 19200 --slave 1 --func 4 --start 5 --count 1"
```
Register 5 contains SOC/SOH combined (SOC = value & 0xFF)

## Version History

- 2026-01-02: Fixed thread leak bug in gen_server.py
- 2026-01-01: Increased max runtime from 2h to 4h
- 2025-12-31: Added 20s AC stabilization delay before charger enable
- 2025-12-30: Added 3-minute cooldown on generator shutdown
- 2025-12-29: Initial PiGenny deployment (hybrid Pi+Olimex system)

---
> Source: [derekja/pigenny](https://github.com/derekja/pigenny) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
