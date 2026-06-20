## wechat-remote-control

> |


# /wechat-remote-control

Determine whether the user wants **login**, **attach**, or **sync**, then follow the steps below.

If the user just says `/wechat-remote-control` with no args, default to **attach**.

**Key principle:** When any step encounters a problem, fix it directly by running commands.
Never ask the user to copy-paste and run commands themselves — handle everything here.

---

## Architecture overview (for Claude's reference)

The bridge uses a **tmux-injection** model, bundled in this skill directory. It drives two
coding agents — **Claude Code** (`claude`) and **OpenAI Codex CLI** (`codex`) — auto-detecting
which one runs in each tmux pane via process ancestry. The multi-session registry records a
`kind` per session, so claude and codex sessions can coexist and be switched with `#sw`.

- **Bridge daemon** (`node src/index.js`): polls ilink WeChat API for messages, injects
  them into the user's tmux-hosted agent session via `tmux send-keys`.
- **Hook server**: listens on Unix socket `/tmp/cc_wechat_hook.sock`. Agent hooks send events
  here via `hook.py`. Claude Code fires PreToolUse / Stop / Notification; Codex fires
  PreToolUse / Stop / UserPromptSubmit (Codex has no Notification event).
- **Response forwarding**: on Stop, the bridge forwards the assistant response to the WeChat
  message it injected. For Claude it parses the transcript JSONL; for Codex it uses the Stop
  payload's `last_assistant_message`, gated on the rollout's latest user message matching what
  was injected. Terminal-initiated responses are NOT forwarded.
- **Auto-approve**: PreToolUse returns `permissionDecision: "allow"` (plus the legacy
  `decision: "approve"`). Both agents honor this — Codex honors `permissionDecision` and
  ignores `decision`, so one payload works for both.
- **All state lives in one directory**: `~/.wechat-remote-control/`
  - `accounts/<accountId>.json` — WeChat credentials
  - `state.json` — tmux target, autoApprove, transcriptPath (legacy single-session)
  - `sessions.json` — multi-session registry (active session + per-tmux-target metadata)
  - `ilink_session.json` — ilink long-poll session cache
  - `get_updates_buf` — ilink sync buffer cursor
  - `bridge.json` / `bridge.pid` / `cc_pid` — daemon metadata
  - `logs/bridge-YYYY-MM-DD.log` — rotated logs (30-day retention)
  - `history.jsonl` — injected messages and forwarded responses

**Critical: process kill safety.** Do NOT use `pgrep -f` or `grep` with bridge path strings
in the same bash command that does other work. Claude Code wraps commands in `bash -c "..."`,
so the pattern matches the shell itself, causing self-termination. Always use the Python
`/proc` scanner shown below, in a **separate** bash call from the start command.

**Environment variables this skill respects:**
- `CLAUDE_CODE_REMOTE=true` — set in cloud sessions. The bridge cannot work in cloud
  (no local tmux), so the skill refuses early with a clear message.
- `CLAUDE_CONFIG_DIR` — relocates `~/.claude/` (undocumented but supported by Claude
  Code; see anthropics/claude-code#3833). Both `detect.py` and the bridge daemon honour
  it when looking up `<config>/projects/<encoded-cwd>/*.jsonl` transcript files.
- `CODEX_HOME` — relocates `~/.codex/` (documented by Codex). Both `detect.py` and the
  bridge daemon honour it when looking up `<home>/sessions/YYYY/MM/DD/rollout-*.jsonl`
  rollout transcripts and when merging hooks into `<home>/hooks.json`.
- `CLAUDECODE=1` — set by Claude Code only. Treated as a hint, NOT a requirement: agent
  kind is determined by process ancestry, since Codex does not set it.

**Helpers:**
- `detect.py` walks `/proc` (Linux) or `ps` (macOS) up from the bash subprocess to find a
  supported agent ancestor (`claude` or `codex`), reports `agent=<kind>`, then verifies it
  lives inside a `tmux list-panes -a` pane. Used by attach Step 1 (preflight) and Step 3
  (state-file writer).

---

## login — Authenticate WeChat account

Run this once before first use, or whenever the bridge reports session expiry.

### Step 1: Check if already logged in

```bash
ls ~/.wechat-remote-control/accounts/*.json 2>/dev/null | head -1
```

If account files exist, note them and ask the user whether they want to re-login
or if this was triggered by a session expiry. If they're just setting up fresh, proceed.

### Step 2: Locate skill directory and ensure dependencies

```bash
echo "=== node/npm prereq ==="
command -v node >/dev/null && node --version || echo "NO_NODE"
command -v npm  >/dev/null && npm  --version || echo "NO_NPM"
echo "=== skill dir ==="
SKILL_DIR=$(find "$HOME" -maxdepth 7 -type f -name "login.js" 2>/dev/null \
  | grep "wechat-remote-control/dist/wechat/login.js" | head -1 \
  | sed 's|/dist/wechat/login.js||')
echo "SKILL_DIR=${SKILL_DIR:-NOT_FOUND}"
```

The QR generator is built into the skill (`dist/wechat/qrcode.js`, zero external deps),
so there is no `npm install` step for QR rendering.

**If `NO_NODE` or `NO_NPM`:** install them yourself — don't push the work onto the user.
Try the system package manager that's available, in this order, and stop on first success:

```bash
if command -v apt-get >/dev/null; then sudo -n apt-get install -y -qq nodejs npm 2>&1 | tail -3 || apt-get install -y -qq nodejs npm 2>&1 | tail -3
elif command -v dnf     >/dev/null; then sudo -n dnf     install -y    nodejs npm 2>&1 | tail -3 || dnf     install -y    nodejs npm 2>&1 | tail -3
elif command -v pacman  >/dev/null; then sudo -n pacman  -S --noconfirm nodejs npm 2>&1 | tail -3 || pacman  -S --noconfirm nodejs npm 2>&1 | tail -3
elif command -v brew    >/dev/null; then brew    install                node       2>&1 | tail -3
else                                    echo "NO_PKG_MGR"
fi
node --version 2>/dev/null && npm --version 2>/dev/null || echo "INSTALL_FAILED"
```

Then verify Node ≥ 18 with `node --version`. If it's older (e.g. Debian 11 ships
Node 12), tell the user to upgrade via nvm or NodeSource — older Node breaks ESM imports
the bridge uses. If `INSTALL_FAILED` or `NO_PKG_MGR`, ask the user to install Node ≥ 18
manually then re-run **login**.

Note the SKILL_DIR value.

### Step 3: Launch background QR-login process (auto-retries on expiry)

The QR is short-lived (~60s). To survive slow scans without forcing the user to start
over, we run the login loop in a **detached background process** that re-requests a
fresh QR whenever the previous one expires, and writes its progress to two files the
foreground polls.

Replace `SKILL_DIR` with the actual path, then run:

```bash
QR_INFO=/tmp/wrc_qr_info.json
QR_RESULT=/tmp/wrc_qr_result.json
rm -f "$QR_INFO" "$QR_RESULT"

nohup node --input-type=module -e "
  import { writeFileSync } from 'fs';
  import { startQrLogin, waitForQrScan } from 'SKILL_DIR/dist/wechat/login.js';
  import { renderTerminalQr } from 'SKILL_DIR/dist/wechat/qrcode.js';
  const renderQr = (url) => { try { return renderTerminalQr(url); } catch { return null; } };

  for (let attempt = 1; attempt <= 5; attempt++) {
    const { qrcodeUrl, qrcodeId } = await startQrLogin();
    const qrcodeText = renderQr(qrcodeUrl);
    writeFileSync('$QR_INFO', JSON.stringify({ qrcodeUrl, qrcodeId, qrcodeText, attempt, ts: Date.now() }));
    try {
      const result = await waitForQrScan(qrcodeId);
      writeFileSync('$QR_RESULT', JSON.stringify({ status:'confirmed', accountId: result.accountId }));
      process.exit(0);
    } catch (e) {
      if (!String(e.message).includes('expired')) {
        writeFileSync('$QR_RESULT', JSON.stringify({ status:'error', error: String(e.message) }));
        process.exit(1);
      }
      // QR expired; loop will request a fresh one.
    }
  }
  writeFileSync('$QR_RESULT', JSON.stringify({ status:'error', error:'too many QR retries' }));
  process.exit(1);
" > /tmp/wrc_qr.log 2>&1 &
echo "QR_LOGIN_PID=$!"
```

### Step 4: Display QR and poll for scan

Wait for the background process to write the QR info, then read it:

```bash
for i in 1 2 3 4 5 6; do [ -f /tmp/wrc_qr_info.json ] && break; sleep 1; done
cat /tmp/wrc_qr_info.json
```

Parse the JSON. **In the same reply**, output `qrcodeText` verbatim as a code block:

```
请在 60 秒内用微信扫描以下二维码登录（过期会自动刷新一张新的）：

​```
<qrcodeText here>
​```
```

If `qrcodeText` is null, show `qrcodeUrl` instead. Remember the `attempt` number — you'll
use it to detect QR rotation.

Then poll for the scan result. If the QR rotates (new `attempt`), re-display the new QR:

```bash
LAST_ATTEMPT=$(python3 -c "import json; print(json.load(open('/tmp/wrc_qr_info.json'))['attempt'])")
for i in $(seq 1 24); do
  if [ -f /tmp/wrc_qr_result.json ]; then
    cat /tmp/wrc_qr_result.json
    exit 0
  fi
  CUR_ATTEMPT=$(python3 -c "import json; print(json.load(open('/tmp/wrc_qr_info.json'))['attempt'])" 2>/dev/null || echo "$LAST_ATTEMPT")
  if [ "$CUR_ATTEMPT" != "$LAST_ATTEMPT" ]; then
    echo "QR_ROTATED"
    exit 0
  fi
  sleep 5
done
echo "POLL_TIMEOUT"
```

Interpret the output:

- **`{"status":"confirmed",...}`** — scan succeeded, proceed to Step 5.
- **`QR_ROTATED`** — old QR expired and a fresh one is ready. Re-read `/tmp/wrc_qr_info.json`,
  display the new `qrcodeText` to the user, and re-run this poll loop.
- **`{"status":"error",...}`** — login failed; report the error and stop.
- **`POLL_TIMEOUT`** — 2 minutes elapsed without scan or rotation; re-run the poll loop
  (the user is still scanning).

### Step 5: Verify and report

```bash
ls ~/.wechat-remote-control/accounts/*.json 2>/dev/null
```

If no files: retry from Step 3. Otherwise confirm login success.

---

## attach — Register this session as WeChat remote target

### Step 1: Pre-flight checks (run all in one bash call)

`detect.py` walks the `/proc` parent chain (or `ps` on macOS) from the bash subprocess
to find the actual `claude` process and verifies it lives inside a tmux pane. This
replaces the old `tmux display-message` check, which was unreliable: `display-message`
returns the *focused* client window, not the pane CC lives in, and the old `||
echo NOT_IN_TMUX` guard never fired (echo always returns 0).

```bash
echo "=== bridge installed ==="
test -f $HOME/.claude/skills/wechat-remote-control/src/index.js && echo "OK" || echo "NOT_FOUND"
echo "=== account ==="
ls ~/.wechat-remote-control/accounts/*.json 2>/dev/null | head -1 || echo "NOT_FOUND"
echo "=== detect ==="
python3 $HOME/.claude/skills/wechat-remote-control/detect.py preflight
echo "=== existing attach ==="
cat ~/.wechat-remote-control/state.json 2>/dev/null || echo "{}"
```

Interpret the `status=...` line from the detect block. Also note the `agent=...` line
(`claude` or `codex`) — it drives the hooks/statusLine steps below.

- **`status=OK`** — proceed. The `agent=...`, `tmux_target=...` and `cc_pid=...` values are
  saved to `/tmp/wrc_detect.json` later in Step 3.
- **`status=REMOTE_SESSION`** — Claude Code is running as a cloud session
  (`CLAUDE_CODE_REMOTE=true`). The bridge uses local tmux + Unix sockets and cannot
  work in cloud. Stop and tell the user to use a local CC instance.
- **`status=NO_AGENT_PROCESS`** — no `claude` or `codex` ancestor was found. Stop and ask
  the user to confirm the agent was started normally (not via wrapper scripts).
- **`status=NO_TMUX`** — `tmux` is not installed. Install it yourself (don't push the
  work to the user), then fall through to the `AGENT_NOT_IN_TMUX` instructions below
  (because the user still needs to restart the agent inside a tmux session — installing
  tmux doesn't move the already-running agent into one):

  ```bash
  if command -v apt-get >/dev/null; then sudo -n apt-get install -y -qq tmux 2>&1 | tail -3 || apt-get install -y -qq tmux 2>&1 | tail -3
  elif command -v dnf     >/dev/null; then sudo -n dnf     install -y    tmux 2>&1 | tail -3 || dnf     install -y    tmux 2>&1 | tail -3
  elif command -v pacman  >/dev/null; then sudo -n pacman  -S --noconfirm tmux 2>&1 | tail -3 || pacman  -S --noconfirm tmux 2>&1 | tail -3
  elif command -v brew    >/dev/null; then brew    install               tmux 2>&1 | tail -3
  else                                    echo "NO_PKG_MGR — please install tmux manually"
  fi
  ```

- **`status=AGENT_NOT_IN_TMUX`** (or after installing tmux above) — **DO NOT silently
  create a tmux session and proceed.** The bridge injects keystrokes into the tmux
  pane where the agent actually lives; injecting into a different pane silently sends keys
  nowhere, and we cannot move an already-running agent into a new tmux session. Substitute
  the detected agent's launch command (`claude` or `codex`) into the message and output:

  > 检测到 agent（Claude Code / Codex）不在 tmux 内运行。WeChat Remote Control 必须把它跑在
  > tmux 里才能转发消息（桥接靠 `tmux send-keys` 把微信消息注入 agent 所在面板）。
  >
  > 请：
  > 1. 退出当前会话（Ctrl-D 或 `/exit`）
  > 2. 启动 tmux：`tmux new -s cc`
  > 3. 在 tmux 里重新运行 `claude`（或 `codex`）
  > 4. 重新执行 `/wechat-remote-control`

  Then stop. Do not proceed.

**Other early stops:**
- **bridge NOT_FOUND** — tell user the bridge is not installed, stop.
- **account NOT_FOUND** — run **login** flow first, then return.

### Step 2: Check for existing attach — prompt for override

If `state.json` shows `active: true` and `injectTarget` exists, show the user:

> "WeChat Remote Control is currently attached to tmux `<session>:<window>.<pane>`
>  (since <attachedAt>). Override with this session? [Y/n]"

Default: **Y** (override). If user says no, stop.

### Step 3: Write state.json, bridge.json, sessions.json

`detect.py json` returns a single JSON blob with `agent`, `cc_pid`, `cwd`, `tmux_target`,
and `transcript`. We feed that into a small writer script that updates the runtime state
files. The transcript path respects `CLAUDE_CONFIG_DIR` / `CODEX_HOME` if set, and the
session entry records `kind` (`claude` or `codex`).

```bash
python3 $HOME/.claude/skills/wechat-remote-control/detect.py json > /tmp/wrc_detect.json
test "$(python3 -c "import json; print(json.load(open('/tmp/wrc_detect.json'))['status'])")" = "OK" || { echo "DETECT_FAILED"; cat /tmp/wrc_detect.json; exit 1; }

python3 - <<'PY'
import json, os, time, datetime, subprocess
info = json.load(open('/tmp/wrc_detect.json'))
d = os.path.expanduser('~/.wechat-remote-control'); os.makedirs(d, exist_ok=True)
target = info['tmux_target']

# state.json (legacy single-session)
json.dump({
    'injectTarget': {
        'session': info['tmux_session'], 'window': info['tmux_window'], 'pane': info['tmux_pane'],
        'attachedAt': int(time.time() * 1000),
    },
    'autoApprove': True,
    'active': True,
    'transcriptPath': info.get('transcript'),
}, open(os.path.join(d, 'state.json'), 'w'), indent=2)

# bridge.json (daemon metadata)
session_id = os.path.basename(info['transcript']).replace('.jsonl', '') if info.get('transcript') else None
json.dump({
    'sessionId': session_id, 'cwd': info['cwd'], 'ccPid': info['cc_pid'],
    'attachedAt': datetime.datetime.now(datetime.timezone.utc).isoformat(),
}, open(os.path.join(d, 'bridge.json'), 'w'), indent=2)

# cc_pid file (used by status.sh)
open(os.path.join(d, 'cc_pid'), 'w').write(str(info['cc_pid']))

# sessions.json — match by tmux target; create entry if new; set active directly.
# Never set active=None — the bridge's auto-pick heuristic can pick the wrong CC
# when multiple sessions share a cwd.
sp = os.path.join(d, 'sessions.json')
sessions = json.load(open(sp)) if os.path.exists(sp) else {'active': None, 'sessions': {}}
sessions.setdefault('sessions', {})

kind = info.get('agent', 'claude')
matched = None
for name, s in sessions['sessions'].items():
    if s.get('tmux') == target:
        matched = name
        s['transcriptPath'] = info.get('transcript')
        s['kind'] = kind
        s['lastSeen'] = int(time.time() * 1000)
        break

if not matched:
    try:
        wname = subprocess.check_output(
            ['tmux', 'display-message', '-t', target, '-p', '#{window_name}'],
            stderr=subprocess.DEVNULL).decode().strip()
    except Exception:
        wname = ''
    SHELLS = {'bash', 'zsh', 'sh', 'fish', 'dash', 'tcsh', 'csh', 'ksh'}
    base = wname if (wname and wname.lower() not in SHELLS and not wname.startswith('[')) else os.path.basename(info['cwd'])
    matched = base
    suffix = 2
    while matched in sessions['sessions']:
        matched = f'{base}-{suffix}'; suffix += 1
    sessions['sessions'][matched] = {
        'tmux': target, 'cwd': info['cwd'],
        'transcriptPath': info.get('transcript'),
        'kind': kind,
        'lastSeen': int(time.time() * 1000),
    }

sessions['active'] = matched
json.dump(sessions, open(sp, 'w'), indent=2)
print(f'OK: target={target} active={matched} ccPid={info["cc_pid"]}')
PY
```

### Step 4: Configure agent hooks (merge-safe)

The hook config file and event set depend on the detected agent:

- **claude** → `$CLAUDE_CONFIG_DIR/settings.json` (default `~/.claude/settings.json`), events
  `PreToolUse` / `Stop` / `Notification`.
- **codex** → `$CODEX_HOME/hooks.json` (default `~/.codex/hooks.json`), events
  `PreToolUse` / `Stop` / `UserPromptSubmit` (Codex has no `Notification` event).

Both files use the same nested `hooks` schema. The writer below **merges** — it never
overwrites other settings or other tools' hook blocks, and it dedupes by our `hook.py`
command so re-running attach is idempotent. It reads the agent kind from the Step-3 detect
blob:

```bash
python3 - <<'PY'
import json, os
info = json.load(open('/tmp/wrc_detect.json'))
kind = info.get('agent', 'claude')
cmd_base = os.path.expanduser('~/.claude/skills/wechat-remote-control/hook.py')

if kind == 'codex':
    cfg_dir = os.environ.get('CODEX_HOME') or os.path.expanduser('~/.codex')
    path = os.path.join(cfg_dir, 'hooks.json')
    events = {'PreToolUse': 'pretooluse', 'Stop': 'stop', 'UserPromptSubmit': 'userpromptsubmit'}
else:
    cfg_dir = os.environ.get('CLAUDE_CONFIG_DIR') or os.path.expanduser('~/.claude')
    path = os.path.join(cfg_dir, 'settings.json')
    events = {'PreToolUse': 'pretooluse', 'Stop': 'stop', 'Notification': 'notification'}

os.makedirs(cfg_dir, exist_ok=True)
cfg = json.load(open(path)) if os.path.exists(path) else {}
hooks = cfg.setdefault('hooks', {})

added = []
for event, arg in events.items():
    command = f'python3 {cmd_base} {arg}'
    groups = hooks.setdefault(event, [])
    # Dedup: skip if any existing handler already runs our hook.py.
    exists = any(
        'wechat-remote-control/hook.py' in (h.get('command') or '')
        for g in groups for h in g.get('hooks', [])
    )
    if not exists:
        groups.append({'matcher': '', 'hooks': [{'type': 'command', 'command': command}]})
        added.append(event)

with open(path, 'w') as f:
    json.dump(cfg, f, indent=2)
print(f'OK kind={kind} file={path} added={added or "none (already present)"}')
PY
```

### Step 5: Configure status line (Claude only)

Codex has no status-line hook mechanism, so **skip this step entirely when `agent=codex`.**
For Claude, check if `statusLine` is already configured; if not (or if the command points
elsewhere), merge it into `$CLAUDE_CONFIG_DIR/settings.json` without overwriting other
settings:

```bash
# Run only when agent=claude.
python3 -c "
import json, os
path = os.path.join(os.environ.get('CLAUDE_CONFIG_DIR') or os.path.expanduser('~/.claude'), 'settings.json')
settings = json.load(open(path)) if os.path.exists(path) else {}
desired = {
    'type': 'command',
    'command': 'bash \$HOME/.claude/skills/wechat-remote-control/status.sh'
}
if settings.get('statusLine') == desired:
    print('ALREADY_SET')
else:
    settings['statusLine'] = desired
    with open(path, 'w') as f:
        json.dump(settings, f, indent=2)
    print('OK')
"
```

If `ALREADY_SET`, skip silently.

### Step 6: Ensure bridge daemon is running (service model)

The bridge is a singleton service. Check if it's already running via PID file before starting.
Never kill a healthy bridge just because attach was called again.

The daemon **self-enforces** singleton at the OS level by binding the hook socket
`/tmp/cc_wechat_hook.sock`: if a live daemon already owns it (even one launched from a
different install dir, e.g. `~/.agents/skills` for Codex vs `~/.claude/skills` for Claude
Code), a second `node src/index.js` detects it and exits immediately. The PID check below
is just an optimization to avoid spawning a doomed process. The daemon writes its own PID
to `bridge.pid` once it successfully binds, so that file always points at the real singleton.

**Check existing daemon:**

```bash
PID_FILE="$HOME/.wechat-remote-control/bridge.pid"
BRIDGE_RUNNING=0
if [ -f "$PID_FILE" ]; then
    STORED_PID=$(cat "$PID_FILE")
    if kill -0 "$STORED_PID" 2>/dev/null; then
        echo "already running PID=$STORED_PID"
        BRIDGE_RUNNING=1
    else
        echo "stale PID file, cleaning up"
        rm -f "$PID_FILE"
    fi
fi
echo "BRIDGE_RUNNING=$BRIDGE_RUNNING"
```

**Only if BRIDGE_RUNNING=0 — start daemon** (separate bash call):

```bash
nohup node $HOME/.claude/skills/wechat-remote-control/src/index.js >> /tmp/cc_wechat_bridge.log 2>&1 &
echo "launched PID=$!"
```

Do NOT write `bridge.pid` here — the daemon writes its own PID once it wins the
singleton. (A redundant launch self-exits without touching `bridge.pid`, so the file
always points at the live daemon.)

**Verify** (after ~3 seconds) — check the live daemon, not the launched PID, since a
redundant launch self-exiting is a *success*, not a failure:

```bash
PID_FILE="$HOME/.wechat-remote-control/bridge.pid"
if [ -f "$PID_FILE" ] && kill -0 "$(cat "$PID_FILE")" 2>/dev/null; then
    echo "running PID=$(cat "$PID_FILE")"
else
    echo "FAILED"
fi
```

If FAILED: read the last 30 lines of `/tmp/cc_wechat_bridge.log` and diagnose.

### Step 7: Report success

```
WeChat Remote Control activated

Agent: <Claude Code | Codex>
Tmux target: <session>:<window>.<pane>
Auto-approve: on
Bridge: running (PID <pid>)
Transcript: <transcript path>

You can leave the terminal now. WeChat messages will be injected into this
agent session via tmux. Responses triggered by WeChat will be forwarded back.

Terminal-initiated responses are NOT forwarded to WeChat.
```

For Claude, a status-line indicator shows bridge state in the terminal; Codex has no
status line, so that indicator is Claude-only.

---

## sync — Show WeChat conversation history

### Step 1: Format and show history

```bash
python3 $HOME/.claude/skills/wechat-remote-control/format_history.py 2>/dev/null \
  || (tail -20 ~/.wechat-remote-control/history.jsonl 2>/dev/null || echo "No WeChat history found.")
```

If no history exists, check the bridge logs:

```bash
tail -50 ~/.wechat-remote-control/logs/bridge-$(date +%Y-%m-%d).log 2>/dev/null | grep -E "INFO|WARN|ERROR" | tail -20
```

### Step 2: Summarize

Give a one-line summary of what happened while the user was away.

---
> Source: [MatrixA/wechat-remote-control](https://github.com/MatrixA/wechat-remote-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
