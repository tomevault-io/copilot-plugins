## agent-local-mail

> >


# agent-mail — a LAN mail server for AI agents

This skill installs **maddy** (a single-binary Go mail server) on a Linux
host and configures it for LAN-only multi-user delivery with a self-signed
TLS cert. The default use case is "let my locally-running AI agent send me
an email" via a mail client like Apple Mail, but it works for any internal
mail scenario.

The reference deployment is a Raspberry Pi reached over SSH (`<target>`),
with mail state on a USB SSD to spare the SD card. The same recipe works
on any Debian/Ubuntu host with `systemd`, `curl`, `tar`, and `zstd`. A
plain-text variant is documented at the end for anyone who genuinely
doesn't want TLS.

## Before you run

Confirm these with the user:

1. **Target host** — SSH alias, e.g. `elf`. Passwordless `sudo` should be
   available (the skill checks).
2. **Mail domain** — what comes after the `@`. The convention is
   `<hostname>.local` (e.g. `magicelf.local`) so that the existing avahi/
   Bonjour `.local` resolution lets every device on the LAN find the
   server without extra DNS config.
3. **State directory** — where mail and the auth DB live. On a Pi this
   should be a path on the SSD/USB (e.g. `/mnt/ssd/maddy`) to spare the
   SD card. Default `/var/lib/maddy` is fine on a non-Pi box.
4. **Accounts to create** — at minimum **one for the user themselves**
   (e.g. `me`, or whatever local-part they like) plus one per agent
   (e.g. `hermes`, `claude`). The user mailbox is what their mail client
   (Apple Mail etc.) logs into to *be* a participant on the LAN — without
   it they'd have to impersonate an agent account, which gets conceptually
   confusing fast. Don't suggest names yourself from the harness context
   (their home directory, their email) — ask.
5. **TLS or plain?** — Default is self-signed TLS, because most modern
   clients (Apple Mail, Outlook, Python's `imaplib` used by Hermes) will
   refuse a plaintext connection or warn obnoxiously. To go fully plain,
   replace the `tls file ...` line in `maddy.conf` with `tls off` and
   drop the `tls://0.0.0.0:993` listener — see the alternate config in
   the "Plain-text mode" section below.

Don't agonise over the domain — it's just a config edit to change later, as
long as no real mail has been received yet.

## Steps

### 1. Recon the target

Over SSH, gather:

```bash
ssh <target> '
  cat /etc/os-release | head -3
  uname -m                                # need aarch64 or x86_64
  hostname
  df -h /                                  # SD card free space
  df -h <state-dir-parent> 2>/dev/null     # SSD free space if not root
  sudo ss -tlnp | grep -E ":(25|143|587|465|993) " || echo "mail ports free"
  dpkg -l | grep -iE "postfix|dovecot|exim|maddy" || echo "no existing mail software"
  systemctl is-active avahi-daemon || echo "no avahi"
'
```

Confirm:

- Architecture is `aarch64` or `x86_64` (maddy ships prebuilt binaries for both).
- Mail ports 25/143/587 are not already taken by Postfix/Exim/Dovecot. On
  Debian, **exim4 is installed by default and binds `127.0.0.1:25`** — this
  must be stopped and disabled before maddy can start.
- Avahi is running (otherwise `.local` won't resolve from LAN clients —
  install `avahi-daemon` or use the host's IP).

### 2. Disable any conflicting mail daemon

```bash
ssh <target> 'sudo systemctl stop exim4 && sudo systemctl disable exim4'
```

(Substitute `postfix`, `dovecot`, etc. if those were found instead. Don't
uninstall — disabling is reversible.)

### 3. Download and install maddy

Find the latest release via the GitHub API. Pick the asset matching the
target arch — `aarch64-linux-musl.tar.zst` for Pi, `x86_64-linux-musl.tar.zst`
for amd64.

```bash
TAG=$(curl -s https://api.github.com/repos/foxcpp/maddy/releases/latest | \
  grep '"tag_name"' | cut -d'"' -f4)
ARCH=$(ssh <target> 'uname -m')   # aarch64 or x86_64
URL="https://github.com/foxcpp/maddy/releases/download/${TAG}/maddy-${TAG#v}-${ARCH}-linux-musl.tar.zst"

ssh <target> "
  set -e
  cd /tmp
  curl -sSL -o maddy.tar.zst '$URL'
  rm -rf /tmp/maddy-extract && mkdir /tmp/maddy-extract
  tar --zstd -xf maddy.tar.zst -C /tmp/maddy-extract
"
```

On the host:

```bash
ssh <target> '
  set -e
  SRC=$(ls -d /tmp/maddy-extract/maddy-*)
  sudo install -m 0755 $SRC/maddy /usr/local/bin/maddy
  getent group maddy >/dev/null || sudo groupadd -r maddy
  id maddy >/dev/null 2>&1 || sudo useradd -r -g maddy -d /var/lib/maddy -s /usr/sbin/nologin maddy
  sudo mkdir -p <state-dir>
  sudo chown maddy:maddy <state-dir>
  sudo chmod 0750 <state-dir>
  sudo mkdir -p /etc/maddy /etc/systemd/system/maddy.service.d
  sudo install -m 0644 $SRC/systemd/maddy.service /etc/systemd/system/maddy.service
'
```

### 3a. Generate a self-signed TLS cert (skip if going plain-text)

Most clients won't talk plaintext. Generate a self-signed cert with SANs
covering every hostname the server is reachable as — LAN mDNS name,
short hostname, LAN IP, plus the Tailscale name and IP if Tailscale is
installed.

```bash
# Detect optional Tailscale identity
TS_FQDN=$(ssh <target> 'tailscale status --json 2>/dev/null | python3 -c "import json,sys;d=json.load(sys.stdin);print(d[\"Self\"][\"DNSName\"].rstrip(\".\"))"' 2>/dev/null || true)
TS_IP=$(ssh <target> 'tailscale ip -4 2>/dev/null' || true)
LAN_IP=$(ssh <target> "hostname -I | awk '{print \$1}'")

SAN="DNS:<hostname>,DNS:<domain>,DNS:localhost,IP:${LAN_IP},IP:127.0.0.1"
[ -n "$TS_FQDN" ] && SAN="${SAN},DNS:${TS_FQDN}"
[ -n "$TS_IP" ]   && SAN="${SAN},IP:${TS_IP}"

ssh <target> "
  sudo mkdir -p /etc/maddy/certs
  sudo openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:P-256 \
    -keyout /etc/maddy/certs/<hostname>.key \
    -out    /etc/maddy/certs/<hostname>.crt \
    -days 3650 -nodes \
    -subj '/CN=<domain>/O=LAN mail/C=GB' \
    -addext 'subjectAltName=${SAN}'
  sudo chown root:maddy /etc/maddy/certs/<hostname>.*
  sudo chmod 0644 /etc/maddy/certs/<hostname>.crt
  sudo chmod 0640 /etc/maddy/certs/<hostname>.key

  # Trust the cert system-wide so Python imaplib (Hermes) accepts it
  sudo cp /etc/maddy/certs/<hostname>.crt /usr/local/share/ca-certificates/maddy-<hostname>.crt
  sudo update-ca-certificates
"
```

Apple Mail will prompt to trust the cert on first connection — that's
expected. If the user has other hosts that need to trust it (e.g. Hermes
running on a different machine), copy `/etc/maddy/certs/<hostname>.crt`
to that host and `update-ca-certificates` (Linux), or import into
Keychain Access (macOS).

### 4. Render and install the config

Take `templates/maddy.conf.template` and substitute:

- `{{MAIL_DOMAIN}}` → the chosen domain (e.g. `magicelf.local`)
- `{{STATE_DIR}}` → the state directory (e.g. `/mnt/ssd/maddy`)
- `{{TLS_CERT}}` → e.g. `/etc/maddy/certs/magicelf.crt`
- `{{TLS_KEY}}`  → e.g. `/etc/maddy/certs/magicelf.key`

Take `templates/override.conf.template` and substitute:

- `{{STATE_DIR}}` → same as above

Push them over and install:

```bash
scp rendered-maddy.conf <target>:/tmp/maddy.conf
scp rendered-override.conf <target>:/tmp/override.conf
ssh <target> '
  sudo install -m 0644 -o root -g maddy /tmp/maddy.conf /etc/maddy/maddy.conf
  sudo install -m 0644 /tmp/override.conf /etc/systemd/system/maddy.service.d/override.conf
  sudo systemctl daemon-reload
  sudo systemctl enable --now maddy
'
```

### 5. Verify it started

```bash
ssh <target> 'sudo systemctl is-active maddy && sudo ss -tlnp | grep maddy'
```

Expect **four** listeners in the default TLS install: 25 (SMTP), 143
(plain IMAP), 587 (submission with STARTTLS), 993 (implicit-TLS IMAP).
In plain-text mode you'll see only three: 25, 143, 587.

If it failed, `sudo journalctl -xeu maddy.service --no-pager | tail -40`
will show why. Common causes — see "Gotchas" below.

### 6. Create user accounts

For each account name (e.g. `claude`, `hermes`):

```bash
PW=$(openssl rand -base64 18 | tr -d '/+=' | cut -c1-20)
ssh <target> "
  printf '%s' '$PW' | sudo -u maddy /usr/local/bin/maddy creds create $USER@$DOMAIN
  sudo -u maddy /usr/local/bin/maddy imap-acct create $USER@$DOMAIN
"
echo "$USER@$DOMAIN  $PW"
```

Both commands are needed: `creds create` sets the password,
`imap-acct create` creates the mailbox. Listing afterwards confirms both:

```bash
ssh <target> 'sudo -u maddy /usr/local/bin/maddy creds list'
ssh <target> 'sudo -u maddy /usr/local/bin/maddy imap-acct list'
```

**Do not persist the plain-text passwords on the host.** Maddy only stores
the bcrypt hash — there is no recovery if the original is lost, but
"write the secret to a file" is not the answer. Instead:

- Print each generated password to the operator's terminal once, at
  install time, with a clear "save this now in your password manager"
  instruction.
- The operator is responsible for putting it somewhere safe (1Password,
  Bitwarden, Keychain Access, etc.) before moving on.
- If they lose it, run `maddyctl creds password <user@domain>` to set
  a new one. No drama.

The skill used to write a backup file to `/root/maddy-credentials.txt` on
the host. That was a mistake — plaintext credentials at rest, even
root-only, is a finding waiting to happen. Don't reintroduce it.

### 7. Smoke test

Two test scripts ship with the skill:

- `smoketest-tls.py` — IMAP4_SSL on 993, SMTP STARTTLS on 587, with strict
  cert verification (the same way Hermes connects). Use this in the
  default TLS install.
- `smoketest.py` — plain IMAP on 143, plain SMTP on 587, no encryption.
  Use this only in plain-text mode.

```bash
scp smoketest-tls.py <target>:/tmp/
ssh <target> 'python3 /tmp/smoketest-tls.py "<receiver-pw>" "<sender-pw>"'
```

A successful run prints `1 message(s) in inbox` and shows the body.
If you get an SSL `CERTIFICATE_VERIFY_FAILED` instead, `update-ca-certificates`
was probably skipped — see step 3a.

### 7a. Set up the user's mail client (Apple Mail or equivalent)

The user needs a real mail client to *be* a participant on the LAN.
For Apple Mail specifically:

1. **Mail → Add Account → Other Mail Account…**
2. Enter the user's mailbox address (e.g. `me@<domain>`) and the password
   you generated.
3. macOS auto-discovery will fail for a `.local` server — that's expected.
   On the manual screen, fill in:
   - Account Type: **IMAP**
   - User Name: the full address (`me@<domain>`), not just the local-part
   - Incoming Mail Server: `<domain>` (or whatever the cert SAN says is
     reachable from this client — `<host>.local` on the LAN, the
     Tailscale FQDN over Tailscale)
   - Outgoing Mail Server: same as incoming
4. **Trust the self-signed cert**: macOS will warn that the identity
   can't be verified. Click **Show Certificate**, tick **Always trust**,
   **Continue**, enter the Mac password. You'll be prompted twice — once
   for IMAP, once for outgoing SMTP. Both same answer.
5. In **Mail → Settings → Accounts → <account> → Server Settings**,
   confirm: incoming port 993 with "Use TLS/SSL" on, outgoing port 587
   with "Use TLS/SSL" on, authentication Password.

Other clients (Thunderbird, K-9, etc.) are equivalent — IMAPS 993,
submission 587 STARTTLS, password auth. They'll all want to first-time
trust the self-signed cert.

#### iOS Mail over Tailscale

A common case: the user wants the mailbox on their iPhone too, so they
can read it when away from home. This works over Tailscale — but only
if the install was done with Tailscale on the target host detected (step
3a includes the Tailscale FQDN and IP in the cert SAN list automatically
when it finds them). Mention this proactively; the user won't think to
ask.

Three things matter:

1. **Hostname**: `<host>.local` is **mDNS/Bonjour and doesn't work over
   Tailscale**. Use the short MagicDNS name (e.g. `<host>`) or the
   tailnet FQDN (`<host>.<tailnet>.ts.net`). Both are in the cert SANs
   from step 3a, so TLS validates over the tailnet just as it does on
   the LAN. The user's email *address* is unchanged — still
   `<user>@<domain>`. Confusing but correct: the address is an
   identifier inside maddy; the hostname is how the client reaches
   the box.

2. **Cert trust on iOS** (the bit users miss): iOS Mail won't offer a
   click-through for an unknown cert the way macOS Mail does. It
   refuses the connection silently. The fix is to install the cert as
   a trust profile:

   ```bash
   # On the user's Mac
   scp <target>:/etc/maddy/certs/<host>.crt ~/Downloads/
   ```

   Then AirDrop the `.crt` to the iPhone (or email it to themselves).
   On the iPhone:

   - **Settings → General → VPN & Device Management** → tap the
     downloaded profile → **Install** → enter passcode → **Install**
     again.
   - **Settings → General → About → Certificate Trust Settings** →
     toggle the new entry **on**. iOS will warn about full trust for
     the root — expected for self-signed.

   The second step is two screens away from the first and is the bit
   people forget. Always mention it.

3. **Adding the account**: **Settings → Mail → Accounts → Add Account
   → Other → Add Mail Account**, with incoming and outgoing host both
   set to the Tailscale name (not `.local`). IMAP port 993, SMTP port
   587, both SSL on, password auth.

If the Tailscale app on the iPhone isn't running, none of this works —
the hostname won't resolve and mail will appear broken. Worth flagging.

### 7b. (Optional) Wire up a Hermes agent

If the user wants their Hermes (Nous Research) agent to use one of the
mail accounts: ship `hermes-env.sh` to the host where Hermes runs and
generate the `.env` block.

```bash
scp hermes-env.sh <hermes-host>:/tmp/
ssh <hermes-host> '/tmp/hermes-env.sh hermes@<domain> "<password>" "<allowed-senders-csv>"'
```

The script prints a ready-to-paste block. Pipe to `~/.hermes/.env` (or
append to whatever Hermes expects). Then start the gateway with
`hermes gateway` per the Hermes docs.

Hermes uses Python's `imaplib`/`smtplib` with default cert verification,
so this only works if the cert from step 3a is in the system trust on
the host running Hermes. If Hermes is on a different machine, copy the
`.crt` over and `update-ca-certificates` there first.

**Hermes onboarding quirks** that bit during the reference deployment:

- The foreground `hermes gateway` log is *terse*. After a successful
  connect it prints `[Email] Connected as hermes@<domain>` and nothing
  else — no "polling started", no per-message activity. That's normal;
  it doesn't mean polling isn't running. The authoritative state files
  are `~/.hermes/gateway_state.json` (look for `"email":
  {"state":"connected"}`) and `~/.hermes/channel_directory.json`
  (registered senders show up here once they email in).
- **First-time onboarding**: when a new sender emails Hermes for the
  first time, Hermes adds them to `channel_directory.json` and replies
  with a `No home channel is set for Email…` message inviting them to
  send `/sethome`. That's not a failure — it's the new-channel pairing
  flow. The user's first agent-replies will appear only *after* they
  reply with `/sethome` (just that, on a line of its own, as the email
  body).
- **Sequence of restarts confuses things.** Hermes marks all existing
  mail as `\Seen` on startup so it doesn't reply to old messages. If
  the user has been sending test mails and restarting Hermes between
  each one, the test mails get swallowed by the "mark all seen" step
  and never reach the agent. Always send the test mail with Hermes
  already running and don't restart in between.

### 8. Tell the user

Report back:

- Server details: hostname:port for each endpoint —
  IMAP `<domain>:993` (implicit TLS), SMTP submission `<domain>:587`
  (STARTTLS), both password auth. Plain `<domain>:143` is also listening
  for debugging.
- Each account's address and password, **printed once to the operator's
  terminal**. Tell the user to save them in their password manager now;
  there is no copy on disk. If they lose one, reset with
  `sudo -u maddy maddyctl creds password <user@domain>`.
- That mail is **LAN-only**: the server refuses to relay to remote
  domains and the `.local` hostname is unreachable from the public
  internet, so the user's existing work/personal email can't reach the
  server. If they want to participate, they use their LAN mailbox in
  their mail client, not their normal address.
- How to add the mailbox to a mail client — point at step 7a.

## Gotchas (learned the hard way)

- **`state_dir` is a maddy directive, not a systemd thing.** Setting
  `WorkingDirectory=` in the systemd unit is not enough — maddy itself
  will still try to `mkdir /var/lib/maddy` unless the top of `maddy.conf`
  has `state_dir <your-path>`. Symptom: `Error: mkdir /var/lib/maddy:
  read-only file system` (because systemd's `ProtectSystem=strict`
  blocks it).
- **`runtime_dir /run/maddy` is required** if you've overridden
  `state_dir`. Maddy's `RuntimeDirectory=maddy` in the unit creates
  `/run/maddy` for you.
- **Exim4 on Debian binds 127.0.0.1:25** by default. Even though it's
  loopback-only, it still prevents maddy from binding `0.0.0.0:25`.
  Stop and disable it.
- **`tls off` is loud.** If you opt into plain-text mode, the maddy binary
  logs "TLS is disabled, this is insecure configuration and should be used
  only for testing!" on every startup and every `maddyctl` invocation.
  This is expected — and another reason the skill defaults to a
  self-signed cert.
- **Hermes silently insists on TLS.** The Nous Research Hermes email
  adapter uses Python's `imaplib.IMAP4_SSL` and `smtplib.SMTP.starttls()`
  with default cert verification and no plain-text override. If the
  system CA bundle doesn't trust maddy's cert, Hermes will fail at
  "IMAP connection failed" on startup with no further detail. Confirm
  with `python3 -c "import imaplib; imaplib.IMAP4_SSL('<host>', 993)"` —
  if that throws `CERTIFICATE_VERIFY_FAILED`, the trust install is
  missing.
- **`maddyctl creds create` reads password from stdin.** Pipe with
  `printf '%s' "$PW" | ...` — don't add a newline (`echo`) or the
  password will include it.
- **Two commands per user**: `creds create` sets the password,
  `imap-acct create` creates the mailbox. Forget the second and IMAP
  login will succeed but `SELECT INBOX` will fail.
- **mDNS scope.** `<host>.local` resolves on the same LAN segment via
  avahi/Bonjour. If a client is on a different VLAN or a guest network,
  resolution will fail — use the host's IP or add a DNS entry.
- **No outbound.** This config has no `target.remote` and no `remote_queue`.
  Mail addressed to any non-local domain is rejected at submission time.
  This is deliberate — adding outbound delivery means dealing with SPF,
  DKIM, DMARC, MTA-STS, reverse DNS, and IP reputation, none of which
  belong on a Pi.
- **Debugging IMAP without contaminating the evidence.** A plain
  `FETCH BODY[...]` or `python imaplib.IMAP4.fetch(i, '(RFC822)')` sets
  `\Seen` on the message as a *side effect* of reading it — standard
  IMAP, but it means a debugging probe will silently alter the state
  you're trying to diagnose. When probing "did Hermes (or anything) read
  this message?", always use a **read-only select** + **`BODY.PEEK`**:

  ```python
  m = imaplib.IMAP4_SSL("<host>", 993)
  m.login("<user>@<domain>", "<password>")
  m.select("INBOX", readonly=True)             # no flag changes on select
  for i in m.search(None, "ALL")[1][0].split():
      m.fetch(i, "(FLAGS BODY.PEEK[HEADER.FIELDS (SUBJECT)])")
  ```

  This is also how to verify "is the message in UNSEEN?" without
  immediately moving it out of UNSEEN yourself.

## Plain-text mode

If you genuinely don't need TLS (single-host, single-user, debugging),
the install is simpler:

- Skip step 3a entirely.
- In `maddy.conf.template`, replace
  `tls file {{TLS_CERT}} {{TLS_KEY}}` with `tls off`.
- In the imap stanza, remove `tls://0.0.0.0:993` so only the plain
  listener remains.

Mail clients that demand TLS won't connect, and maddy will log "TLS is
disabled" loudly on every startup and every `maddyctl` invocation. Use
`smoketest.py` (the plain version) to exercise this mode.

## Files in this skill

- `SKILL.md` — this file.
- `templates/maddy.conf.template` — maddy config with placeholders
  `{{MAIL_DOMAIN}}`, `{{STATE_DIR}}`, `{{TLS_CERT}}`, `{{TLS_KEY}}`.
- `templates/override.conf.template` — systemd drop-in to point
  `WorkingDirectory` and `ReadWritePaths` at the chosen state dir.
- `smoketest-tls.py` — TLS-strict end-to-end test (993 + 587 STARTTLS).
- `smoketest.py` — plaintext end-to-end test (143 + 587), for the
  plain-text mode above.
- `hermes-env.sh` — emits a `.env` block for the Hermes email gateway.

## Reversal

If the user changes their mind:

```bash
ssh <target> '
  sudo systemctl disable --now maddy
  sudo rm -f /usr/local/bin/maddy
  sudo rm -rf /etc/maddy /etc/systemd/system/maddy.service /etc/systemd/system/maddy.service.d
  sudo systemctl daemon-reload
  sudo userdel maddy 2>/dev/null
  sudo groupdel maddy 2>/dev/null
  # Remove the cert trust we installed at step 3a
  sudo rm -f /usr/local/share/ca-certificates/maddy-*.crt
  sudo update-ca-certificates --fresh
  # Mail data stays at <state-dir> until you delete it manually.
  sudo systemctl enable --now exim4   # restore previous mail handler if any
'
```

On any mail-client machine that trusted the self-signed cert (Apple Mail
will have added it to the login keychain), the trust entry needs removing
from there too — that one's manual via Keychain Access on macOS, or
`update-ca-certificates --fresh` after removing from
`/usr/local/share/ca-certificates/` on Linux.

---
> Source: [ohnotnow/agent-local-mail](https://github.com/ohnotnow/agent-local-mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
