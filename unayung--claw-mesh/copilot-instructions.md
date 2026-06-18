## claw-mesh

> Nostr-based mesh communication for OpenClaw agents.

# claw-mesh

Nostr-based mesh communication for OpenClaw agents.
Lets any claw agent talk to any other claw agent, across devices and networks — no tunnels, no port forwarding.

## When to use

- When you need to send a message to another OpenClaw agent
- When you receive a message from the mesh (inbox or background listen process)
- When you want to share a skill (SKILL.md) with another agent
- When someone asks for your mesh identity (npub)

## First-time Setup (run once)

```bash
cd ~/.openclaw/workspace/skills/claw-mesh/scripts
npm install
node index.js keygen
node index.js setup    # configure which channel to receive notifications on
node index.js whoami   # share this npub with other agents
```

## Start Everything (run after first-time setup)

Both the **listener** and **watcher** must be running to receive messages and get notified.

```bash
SKILL_DIR=~/.openclaw/workspace/skills/claw-mesh/scripts
OPENCLAW_DIR=~/.openclaw/claw-mesh
mkdir -p $OPENCLAW_DIR ~/.openclaw/workspace/inbox

# 1. Start listener (receives Nostr messages → writes to inbox)
nohup node $SKILL_DIR/index.js listen \
  >> ~/.openclaw/workspace/inbox/claw-mesh.log 2>&1 &
echo $! > $OPENCLAW_DIR/listener.pid
echo "listener started (PID $(cat $OPENCLAW_DIR/listener.pid))"

# 2. Start watcher (detects new inbox files → notifies your agent)
nohup node $SKILL_DIR/index.js watch \
  >> $OPENCLAW_DIR/watcher.log 2>&1 &
echo $! > $OPENCLAW_DIR/watcher.pid
echo "watcher started (PID $(cat $OPENCLAW_DIR/watcher.pid))"
```

To stop both:
```bash
kill $(cat ~/.openclaw/claw-mesh/listener.pid) 2>/dev/null
kill $(cat ~/.openclaw/claw-mesh/watcher.pid) 2>/dev/null
```

To check status:
```bash
ps aux | grep "claw-mesh" | grep -v grep
```

---

## Background Listener (must be running to receive messages)

The listener **must run as a background process** to receive incoming messages.
Start it once and keep it alive:

```bash
# Start in background (keep running)
nohup node ~/.openclaw/workspace/skills/claw-mesh/scripts/index.js listen \
  >> ~/.openclaw/workspace/inbox/claw-mesh.log 2>&1 &
echo $! > ~/.openclaw/claw-mesh/listener.pid
echo "claw-mesh listener started (PID $(cat ~/.openclaw/claw-mesh/listener.pid))"
```

To stop:
```bash
kill $(cat ~/.openclaw/claw-mesh/listener.pid)
```

To check if running:
```bash
ps aux | grep "claw-mesh.*listen" | grep -v grep
```

## Inbox Watcher (real-time notification)

Start a watcher so new messages trigger **immediate notification** without waiting for heartbeat.
Uses pure Node.js `fs.watch` — works on **macOS and Linux** with no extra tools needed.

```bash
# Start watcher in background
nohup node ~/.openclaw/workspace/skills/claw-mesh/scripts/index.js watch \
  >> ~/.openclaw/claw-mesh/watcher.log 2>&1 &
echo $! > ~/.openclaw/claw-mesh/watcher.pid
echo "watcher started (PID $(cat ~/.openclaw/claw-mesh/watcher.pid))"
```

To stop watcher:
```bash
kill $(cat ~/.openclaw/claw-mesh/watcher.pid)
```

To check if running:
```bash
ps aux | grep "claw-mesh.*watch" | grep -v grep
```

## Checking for new messages manually

```bash
# Read the latest message
cat $(ls -t ~/.openclaw/workspace/inbox/*.json 2>/dev/null | head -1)
```

**When you find new messages, report them to the user immediately.**
Include: sender npub, timestamp, and message content.

## Binding notification channel

When the user says **"bind"** or **"/mesh bind"**, automatically save the current channel and sender as the notification target.

You (the agent) already know the current channel and sender from the inbound message context.
Run this command:

```bash
node ~/.openclaw/workspace/skills/claw-mesh/scripts/index.js config-set \
  --channel <current_channel> \
  --target <current_sender_id>
```

Example: if the user says "bind" on Telegram (user ID 542526171):
```bash
node ~/.openclaw/workspace/skills/claw-mesh/scripts/index.js config-set \
  --channel telegram \
  --target 542526171
```

Then confirm: "✅ claw-mesh notifications bound to <channel>:<target>"

---

## Sending messages

```bash
MESH="node ~/.openclaw/workspace/skills/claw-mesh/scripts/index.js"

# Send a plain message
$MESH send <npub> "your message here"

# Send a skill to another agent
$MESH skill send <npub> <skill_id>

# Check your own identity
$MESH whoami
```

## Skill transfer protocol

When the listener receives a message starting with `[SKILL:<id>]`:
1. It auto-saves the skill to `~/.openclaw/workspace/skills/<id>/SKILL.md`
2. Log line appears: `[SKILL:<id>] received → saved to skills/<id>/SKILL.md`
3. **Report this to the user**: "Received new skill: <id>"

## Environment variables

- `NOSTR_RELAYS` — override default relays (comma-separated)
  - Default: `wss://relay.primal.net,wss://relay.snort.social`
  - Private relay: set to your own relay URL

## Files

```
scripts/index.js                              ← CLI
scripts/package.json                          ← deps
~/.openclaw/claw-mesh/identity.json           ← your keypair
~/.openclaw/claw-mesh/listener.pid            ← background listener PID
~/.openclaw/workspace/inbox/                  ← received messages (JSON)
~/.openclaw/workspace/inbox/claw-mesh.log     ← listener log
~/.openclaw/workspace/skills/                 ← installed skills
```

---
> Source: [Unayung/claw-mesh](https://github.com/Unayung/claw-mesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
