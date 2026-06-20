## phantun-wireguard-tunnel-skill

> Deploy a Phantun (UDP→fake-TCP obfuscator) + WireGuard tunnel between a home router (OpenWrt or MikroTik RouterOS 7+ with a container runtime) and a Debian/Ubuntu VPS. Use when the user asks to set up such a tunnel, when their ISP throttles/blocks UDP, when WireGuard handshake is unstable due to QoS, when they want to disguise a self-hosted WG egress as TCP, or when they want to deploy multiple parallel phantun tunnels to different VPSs. Bridges the WG endpoint into daed/OpenClash/Mihomo/sing-box.


# Phantun + WireGuard Tunnel: Home Router ⇄ Linux VPS

A battle-tested deployment guide covering the **non-obvious gotchas** that consume hours when followed naively from upstream READMEs. Covers two router platforms:

- **OpenWrt 24.10+** (APK + fw4 / nftables) — same kernel, single netns, simplest
- **MikroTik RouterOS 7.4+** with container runtime — multi-netns, has extra gotchas around routing and firewall

Reference plan file: `~/.claude/plans/openwrt-daed-vps-debian13-phantun-wireg-wise-bird.md`.

## When to use this skill

The user wants to:
- Build a WG tunnel where the WG UDP is wrapped as fake TCP (Phantun) to bypass ISP UDP QoS / blocking
- Self-host a WG egress on a Linux VPS reachable from a Chinese-network home router
- Bridge that WG endpoint into a proxy framework:
  - OpenWrt: **daed** (via sing-box SOCKS5 because daed lacks WG outbound), **OpenClash / Mihomo** (native WG support), or **sing-box** alone (transparent proxy)
  - RouterOS: **RouterOS native WireGuard** (simplest) or **Mihomo** running in another container
- Run phantun-client inside a **RouterOS container** (x86_64 or aarch64) because they don't want an extra OpenWrt device
- Deploy **multiple parallel tunnels** to different VPSs (one container per VPS)
- Diagnose an existing phantun+WG setup that "should work but doesn't"

## Architecture

```
LAN client ──► OpenWrt proxy framework (daed/OpenClash/sing-box)
                 ↓ as WG client
              UDP 127.0.0.1:4567  ──► phantun-client (TUN + raw socket)
                                        ↓ fake TCP packet
                                      pppoe-wan / eth-wan
                                        ↓ encrypted-looking TCP
                                      VPS public IP : 4567
                                        ↓
                                      phantun-server (TUN + raw socket)
                                        ↓ decapsulated UDP
                                      127.0.0.1 : 51820
                                        ↓
                                      WireGuard kernel wg0  (10.200.0.1)
                                        ↓ NAT MASQUERADE
                                      Internet
```

**Layering rationale**: Phantun does only UDP↔TCP shape-shifting (no encryption). WG provides crypto/auth. Two WG keypairs are needed (server + client). One Phantun keypair is implicit (Phantun has no auth).

## Critical gotchas — read this section first, every time

These five issues caused the longest debugging cycles:

### G1. VPS phantun-server REQUIRES a DNAT rule

`phantun-server` reads packets from its TUN interface, **not** from the eth0 raw socket directly. Without DNAT, incoming TCP SYNs on eth0:4567 never reach phantun-server, no SYN-ACK is ever generated, and connections silently time out. **You will see SYNs in `tcpdump -i eth0` but no replies, and journalctl will be silent — no error.**

Required rule (substitute your phantun TCP port and tun-peer IP):
```bash
iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 4567 -j DNAT --to-destination 192.168.200.2
```

`192.168.200.2` is `phantun-server --tun-peer` (default). Confirm via `phantun-server --help`.

**Misleading observation**: Testing the VPS port from a Windows machine via `bash /dev/tcp/<vps>/4567` may show "connection succeeded" even when phantun is broken — middlebox TCP optimization on the *client* side fakes a SYN-ACK locally. Always verify the SYN-ACK is actually emitted from VPS via `tcpdump -i eth0`.

### G2. OpenWrt 24.10+ uses APK and fw4 (nftables) — old guides break

- `opkg` is gone, use `apk add <pkg>`
- Default firewall is **fw4** (nftables `inet fw4` table), policy is `forward DROP`
- iptables-nft compat layer works but **fw4 reload wipes nft rules in the `inet fw4` table** (e.g. our `nft insert rule inet fw4 forward iifname "tun-ph" accept` was lost on every firewall reload)
- iptables rules in legacy `ip filter` / `ip nat` tables (e.g. `OUTPUT` chain RST drop, MASQUERADE in `POSTROUTING`) DO survive fw4 reload (different table)

**Fix**: register the phantun TUN as a UCI firewall zone so fw4 includes it natively:
```sh
uci set firewall.phantun=zone
uci set firewall.phantun.name='phantun'
uci set firewall.phantun.input='ACCEPT'
uci set firewall.phantun.output='ACCEPT'
uci set firewall.phantun.forward='ACCEPT'
uci set firewall.phantun.mtu_fix='1'
uci add_list firewall.phantun.device='tun-ph'

uci set firewall.phantun_to_wan=forwarding
uci set firewall.phantun_to_wan.src='phantun'
uci set firewall.phantun_to_wan.dest='wan'
uci commit firewall && /etc/init.d/firewall reload
```

The wan zone's existing `masq=1` will NAT outgoing traffic — no need for a separate iptables MASQUERADE.

OpenWrt may also need `apk add kmod-ipt-nat` so iptables-nft `MASQUERADE` target works (only relevant if you don't use the UCI zone approach above).

### G3. Phantun OpenWrt binary must be musl, not gnu

OpenWrt uses musl libc. The glibc binary fails with `not found` (missing dynamic loader). Always use `phantun_x86_64-unknown-linux-musl.zip` (or the `aarch64-...-musl` variant for ARM routers). VPS Debian/Ubuntu can use either gnu or musl.

Verify: after install, `/usr/bin/phantun-client --help | head -2` should print Usage. If it prints `not found`, you grabbed the wrong build.

### G4. Phantun release zip filenames are NOT `client` / `server`

The zip extracts to `phantun_client` and `phantun_server` (with underscore prefix). Old guides that say `mv client /usr/local/bin/...` will fail. Use the actual filenames.

Also: **busybox does not ship `install`**. Use `cp + chmod 755` on OpenWrt:
```sh
cp phantun_client /usr/bin/phantun-client && chmod 755 /usr/bin/phantun-client
```

### G5. `dial_mode` semantics in dae/daed

The dial_mode setting controls whether dae passes IP or hostname to upstream proxies. It affects whether the upstream proxy needs its own DNS resolver:

| `dial_mode` | What dae sends to proxy | Sniffing | Domain rules work? | Upstream needs DNS? |
|---|---|---|---|---|
| `ip` | IP only | OFF | ❌ | ❌ |
| `domain` | hostname | ON | ✅ | ✅ |
| `domain+` | hostname (more aggressive sniff) | ON | ✅ | ✅ |
| `domain++` | hostname (force) | ON | ✅ | ✅ |

**Common confusion**: `domain+` does NOT pass IP — only `ip` does. If you want to remove sing-box's DNS module, you must switch dae to `dial_mode: ip` AND convert all `domain(...)` routing rules to `dip(...)` IP-based rules. Usually not worth it; keep upstream DNS.

### G6. RouterOS container — **must** add a route on the host for the TUN_PEER subnet

Only applicable to **RouterOS + container** deployments, not OpenWrt.

Phantun-client crafts fake-TCP packets with **source IP = `--tun-peer`** (e.g. `192.168.201.1`). In OpenWrt this is fine because the phantun TUN and the router are in the same netns — the router knows how to route `192.168.201.0/24`. In RouterOS the phantun process runs in a container netns; the host RouterOS has **no route** to the TUN peer subnet.

Symptom: outbound SYN reaches the VPS fine (NAT-masqueraded), VPS replies with SYN-ACK, conntrack on RouterOS reverses the NAT → destination becomes `192.168.201.1` → **RouterOS routing table has no entry for that subnet → packet black-holed**. Phantun client times out on `Waiting for SYN+ACK`. conntrack shows the flow stuck in `syn-recv` state forever.

Fix (one line per container, different subnet per container):
```routeros
/ip/route/add dst-address=192.168.201.0/24 gateway=<container-LAN-IP> comment=phantun-return
```

This is **THE #1 reason a RouterOS container phantun deployment "looks right but doesn't work"**. Not a firewall issue — a routing issue. Don't waste hours on firewall rules before checking this.

### G7. RouterOS container — iptables OUTPUT DROP RST must be **inside the container netns**

Again RouterOS-specific. When the VPS's SYN-ACK arrives at the container netns, the container's kernel TCP stack (no matching socket) instinctively emits a **RST** back to the VPS. That RST kills phantun's fake-TCP handshake. The skill's classic `iptables -A OUTPUT -p tcp -d <VPS_IP> --dport <VPS_PORT> --tcp-flags RST RST -j DROP` must run **inside the container's network namespace**, not on the RouterOS host.

Practical consequence: your container image **must include `iptables`** (nft or legacy backend, either works), and the entrypoint must install the rule before exec'ing phantun-client. A `scratch + busybox-only` image will not work on RouterOS — it will fail this rule installation and the handshake will silently time out.

Recommended image base: `alpine-minirootfs` + `apk add iptables iptables-legacy`. At ~10 MB total it's the smallest practical container that can host phantun + the required iptables rule.

Entrypoint should try nft first, fall back to legacy:
```sh
install_rst_drop() {
    local IPT="$1"
    $IPT -C OUTPUT -p tcp -d "$VPS_IP" --dport "$VPS_PORT" --tcp-flags RST RST -j DROP 2>/dev/null && return 0
    $IPT -A OUTPUT -p tcp -d "$VPS_IP" --dport "$VPS_PORT" --tcp-flags RST RST -j DROP
}
install_rst_drop iptables-nft || install_rst_drop iptables-legacy || echo "WARN: could not install RST DROP rule"
```

### G8. Mihomo WireGuard proxy requires `remote-dns-resolve: true` in China networks

The skill's C-B section already shows this, but it's worth elevating to a gotcha because many users miss it.

**WireGuard is an L3 proxy**. Unlike Hysteria2 / VMess / Trojan / Shadowsocks (L7, pass domain name to the server), Mihomo must resolve `www.google.com` to an IP **before** encapsulating the TCP packet into the WG tunnel. By default this resolution uses Mihomo's global `dns: nameserver:` — in a Chinese network this is AliDNS / TencentDNS / ISP DNS, all of which return poisoned IPs for sensitive domains.

Symptom: Hysteria2 etc. to the same VPS work fine (sniffer extracts SNI, domain forwarded to server), but WG proxy can't open Google/YouTube/etc. Mihomo `/proxies/<name>/delay?url=https://www.google.com/generate_204` times out while `delay?url=https://www.gstatic.com/generate_204` works (because gstatic.com is on a CDN with less aggressive pollution).

Fix (per WG proxy, not global):
```yaml
  - name: "wg-vps"
    type: wireguard
    ...
    remote-dns-resolve: true
    dns: [ "8.8.8.8", "8.8.4.4" ]
```
This tells Mihomo: when dialing this proxy, send the DNS query **through the WG tunnel itself** to 8.8.8.8 / 8.8.4.4 (VPS-side resolution → clean). This is a **per-proxy** setting — other non-WG proxies on the same Mihomo are unaffected.

## Phase A — VPS (Debian 13 / Ubuntu)

### A1. Probe environment first

```bash
ssh root@<vps>
cat /etc/os-release | head -2
ip route | head -1                   # note the WAN interface (typically eth0)
ss -ltnp | grep -E ':(4567|51820)'   # ensure ports free
sysctl net.ipv4.ip_forward
```

### A2. Install + enable forwarding

```bash
apt update && apt install -y wireguard-tools unzip iptables-persistent
echo 'net.ipv4.ip_forward=1' > /etc/sysctl.d/99-wg-phantun.conf
sysctl --system
```

### A3. Generate WG keypair, fetch Phantun

```bash
umask 077; mkdir -p /etc/wireguard && cd /etc/wireguard
wg genkey | tee server.key | wg pubkey > server.pub
# Decide: client keypair generated on VPS (convenient) or on OpenWrt (more secure)
wg genkey | tee client.key | wg pubkey > client.pub

cd /tmp
wget https://github.com/dndx/phantun/releases/download/v0.8.1/phantun_x86_64-unknown-linux-gnu.zip -O p.zip
unzip p.zip
install -m 0755 phantun_server /usr/local/bin/phantun-server
install -m 0755 phantun_client /usr/local/bin/phantun-client
rm -f p.zip phantun_server phantun_client
phantun-server --version  # confirms 0.8.1
```

### A4. WireGuard config `/etc/wireguard/wg0.conf`

```ini
[Interface]
Address = 10.200.0.1/24
ListenPort = 51820
PrivateKey = <contents of /etc/wireguard/server.key>
MTU = 1300
PostUp   = iptables -A INPUT -p udp --dport 51820 ! -s 127.0.0.1 -j DROP
PostDown = iptables -D INPUT -p udp --dport 51820 ! -s 127.0.0.1 -j DROP
PostUp   = iptables -t nat -A POSTROUTING -s 10.200.0.0/24 -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 10.200.0.0/24 -o eth0 -j MASQUERADE
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT
PostUp   = iptables -A FORWARD -o wg0 -j ACCEPT
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT
PostDown = iptables -D FORWARD -o wg0 -j ACCEPT

[Peer]
PublicKey = <contents of /etc/wireguard/client.pub>
AllowedIPs = 10.200.0.2/32
PersistentKeepalive = 25
```

The `INPUT DROP` for non-localhost UDP 51820 is critical: without it, attackers could bypass phantun and hit WG directly.

### A5. systemd unit `/etc/systemd/system/phantun-server.service`

```ini
[Unit]
Description=Phantun Server (TCP->UDP obfuscator for WireGuard)
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
Environment=RUST_LOG=info
# G1: DNAT incoming TCP to TUN peer — REQUIRED, server is silent without this
ExecStartPre=-/usr/sbin/iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 4567 -j DNAT --to-destination 192.168.200.2
# Drop kernel RST that would fight phantun's fake-TCP
ExecStartPre=-/usr/sbin/iptables -A OUTPUT -p tcp --sport 4567 --tcp-flags RST RST -j DROP
# Allow FORWARD on phantun TUN (Debian default FORWARD policy may be DROP)
ExecStartPre=-/usr/sbin/iptables -A FORWARD -i tun0 -j ACCEPT
ExecStartPre=-/usr/sbin/iptables -A FORWARD -o tun0 -j ACCEPT
# MASQUERADE phantun TUN return path
ExecStartPre=-/usr/sbin/iptables -t nat -A POSTROUTING -s 192.168.200.0/24 -o eth0 -j MASQUERADE
ExecStart=/usr/local/bin/phantun-server --local 4567 --remote 127.0.0.1:51820 --tun tun0 --tun-local 192.168.200.1 --tun-peer 192.168.200.2
ExecStopPost=-/usr/sbin/iptables -t nat -D PREROUTING -p tcp -i eth0 --dport 4567 -j DNAT --to-destination 192.168.200.2
ExecStopPost=-/usr/sbin/iptables -D OUTPUT -p tcp --sport 4567 --tcp-flags RST RST -j DROP
ExecStopPost=-/usr/sbin/iptables -D FORWARD -i tun0 -j ACCEPT
ExecStopPost=-/usr/sbin/iptables -D FORWARD -o tun0 -j ACCEPT
ExecStopPost=-/usr/sbin/iptables -t nat -D POSTROUTING -s 192.168.200.0/24 -o eth0 -j MASQUERADE
Restart=always
RestartSec=3
AmbientCapabilities=CAP_NET_ADMIN CAP_NET_RAW

[Install]
WantedBy=multi-user.target
```

### A6. Enable, start, persist, verify

```bash
systemctl daemon-reload
systemctl enable --now phantun-server wg-quick@wg0
netfilter-persistent save                    # persist iptables across reboots
wg show                                      # interface up, listening :51820
journalctl -u phantun-server -n 5 --no-pager # should say "Listening on 4567"
iptables -t nat -nvL PREROUTING | grep 4567  # confirm DNAT rule present
```

## Phase B — OpenWrt (modern fork: 24.10+, APK + fw4)

### B1. Probe environment

```sh
ssh root@<openwrt>
cat /etc/openwrt_release | grep DISTRIB_RELEASE   # >= 24.10 means APK + fw4
uname -m                                          # confirm arch (x86_64 / aarch64)
which apk opkg                                    # 24.10+: apk only
nft list table inet fw4 2>/dev/null | head -3     # fw4 active?
df -h /                                           # need ~10 MB
```

### B2. Install dependencies

For OpenWrt 24.10+ with APK:
```sh
apk add unzip wget tcpdump kmod-tun kmod-ipt-nat iptables-nft
```

For older opkg systems: `opkg install ...` with same package names (mostly).

### B3. Install Phantun musl binary

```sh
cd /tmp
wget https://github.com/dndx/phantun/releases/download/v0.8.1/phantun_x86_64-unknown-linux-musl.zip -O p.zip
# (use phantun_aarch64-unknown-linux-musl.zip on ARM)
unzip p.zip
cp phantun_client /usr/bin/phantun-client && chmod 755 /usr/bin/phantun-client
cp phantun_server /usr/bin/phantun-server && chmod 755 /usr/bin/phantun-server
rm -f p.zip phantun_client phantun_server
/usr/bin/phantun-client --version       # 0.8.1
```

### B4. procd init `/etc/init.d/phantun-client`

Substitute `<VPS_IP>` and `<WAN_IF>` (typically `pppoe-wan` for Chinese ISP, or `eth1`/`wan`):

```sh
#!/bin/sh /etc/rc.common
# phantun client: WG UDP -> fake TCP to VPS

USE_PROCD=1
START=85
STOP=15

VPS_IP="<VPS_IP>"
VPS_PORT="4567"
TUN_IF="tun-ph"
TUN_LOCAL="192.168.201.2"
TUN_PEER="192.168.201.1"

start_service() {
    iptables -C OUTPUT -p tcp -d ${VPS_IP} --dport ${VPS_PORT} --tcp-flags RST RST -j DROP 2>/dev/null || \
        iptables -A OUTPUT -p tcp -d ${VPS_IP} --dport ${VPS_PORT} --tcp-flags RST RST -j DROP

    procd_open_instance
    procd_set_param command /usr/bin/phantun-client \
        --local 127.0.0.1:4567 \
        --remote ${VPS_IP}:${VPS_PORT} \
        --tun ${TUN_IF} \
        --tun-local ${TUN_LOCAL} \
        --tun-peer ${TUN_PEER}
    procd_set_param env RUST_LOG=info
    procd_set_param respawn 3600 5 0
    procd_set_param stdout 1
    procd_set_param stderr 1
    procd_close_instance
}

stop_service() {
    iptables -D OUTPUT -p tcp -d ${VPS_IP} --dport ${VPS_PORT} --tcp-flags RST RST -j DROP 2>/dev/null
}
```

```sh
chmod +x /etc/init.d/phantun-client
/etc/init.d/phantun-client enable
/etc/init.d/phantun-client start
ip addr show tun-ph                  # should be UP with 192.168.201.2
logread -e phantun | tail -5         # "Created TUN device tun-ph"
```

### B5. UCI firewall zone (G2 fix)

This is **mandatory** on OpenWrt 24.10+ because fw4's `forward` policy is DROP. Without this, all phantun traffic is dropped silently.

```sh
uci set firewall.phantun=zone
uci set firewall.phantun.name='phantun'
uci set firewall.phantun.input='ACCEPT'
uci set firewall.phantun.output='ACCEPT'
uci set firewall.phantun.forward='ACCEPT'
uci set firewall.phantun.mtu_fix='1'
uci add_list firewall.phantun.device='tun-ph'

uci set firewall.phantun_to_wan=forwarding
uci set firewall.phantun_to_wan.src='phantun'
uci set firewall.phantun_to_wan.dest='wan'
uci commit firewall
/etc/init.d/firewall reload

# verify fw4 picked it up
nft list chain inet fw4 forward | grep tun-ph
```

## Phase B-R — RouterOS 7.4+ with container (alternative to OpenWrt)

Use this instead of Phase B when the home router is **MikroTik RouterOS** and you want phantun-client running inside a RouterOS container.

### B-R1. Confirm RouterOS supports containers

```routeros
/system/resource/print                          ;# note architecture-name (x86_64 / arm64)
/system/package/print where name="container"    ;# must be "installed"
```

If container package missing: download the `extra-packages` for your RouterOS version, drag the `container-*.npk` into Files, reboot. Then enable container mode (requires physical reset button press or second-confirm):
```routeros
/system/device-mode/update container=yes
```

### B-R2. Build the Docker image with iptables **without needing Docker**

Phantun-client by itself is a static musl binary (works on scratch), but on RouterOS the container **must also have iptables** to install the OUTPUT RST DROP rule (G7). The clean approach is an Alpine-minirootfs-based image. If you have Docker + buildx, build normally. If you don't, you can construct a Docker v1.2 image tar by hand on any Linux box using `apk-tools-static`:

```bash
WORK=$HOME/phantun-image && mkdir -p "$WORK" && cd "$WORK"
ALPINE_VER="3.19"
ARCH="x86_64"   # or aarch64 for ARM MikroTik
PHANTUN_VER="v0.8.1"

# 1. minirootfs (discover latest 3.19.x)
MR=$(wget -qO- https://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VER}/releases/${ARCH}/ \
     | grep -oE 'alpine-minirootfs-[0-9]+\.[0-9]+\.[0-9]+-'${ARCH}'\.tar\.gz' | sort -V | tail -1)
wget -q https://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VER}/releases/${ARCH}/$MR -O minirootfs.tar.gz

# 2. apk-tools-static (for offline install into rootfs, no root needed)
AK=$(wget -qO- https://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VER}/main/${ARCH}/ \
     | grep -oE 'apk-tools-static-[0-9.]+-r[0-9]+\.apk' | sort -V | tail -1)
wget -q https://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VER}/main/${ARCH}/$AK -O apk-static.apk
mkdir -p apk-x && tar -xzf apk-static.apk -C apk-x
APK_STATIC=$(find apk-x -name apk.static | head -1) && chmod +x "$APK_STATIC"

# 3. phantun binary
wget -q https://github.com/dndx/phantun/releases/download/${PHANTUN_VER}/phantun_${ARCH}-unknown-linux-musl.zip -O p.zip
python3 -c "import zipfile; zipfile.ZipFile('p.zip').extractall('.')"

# 4. rootfs: minirootfs + iptables via apk --root
mkdir rootfs && tar -xzf minirootfs.tar.gz -C rootfs
"$APK_STATIC" --root rootfs --no-cache --arch ${ARCH} \
    -X https://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VER}/main \
    -X https://dl-cdn.alpinelinux.org/alpine/v${ALPINE_VER}/community \
    --allow-untrusted add iptables iptables-legacy
# You may see "chroot: Operation not permitted" for busybox triggers — harmless, we're not root.

install -m 755 phantun_client rootfs/usr/local/bin/phantun-client
cat > rootfs/entrypoint.sh <<'EOS'
#!/bin/sh
set -e
: "${VPS_IP:?VPS_IP is required}"
VPS_PORT="${VPS_PORT:-4567}"
LOCAL_PORT="${LOCAL_PORT:-4567}"
TUN_IF="${TUN_IF:-tun-ph}"
TUN_LOCAL="${TUN_LOCAL:-192.168.201.2}"
TUN_PEER="${TUN_PEER:-192.168.201.1}"
export RUST_LOG="${RUST_LOG:-info}"
install_rst_drop() {
    $1 -C OUTPUT -p tcp -d "$VPS_IP" --dport "$VPS_PORT" --tcp-flags RST RST -j DROP 2>/dev/null && return 0
    $1 -A OUTPUT -p tcp -d "$VPS_IP" --dport "$VPS_PORT" --tcp-flags RST RST -j DROP
}
install_rst_drop iptables-nft 2>/dev/null && echo "[entrypoint] RST DROP via nft" \
    || { install_rst_drop iptables-legacy 2>/dev/null && echo "[entrypoint] RST DROP via legacy"; } \
    || echo "[entrypoint] WARN: no RST DROP"
exec /usr/local/bin/phantun-client --local "0.0.0.0:${LOCAL_PORT}" --remote "${VPS_IP}:${VPS_PORT}" \
    --tun "${TUN_IF}" --tun-local "${TUN_LOCAL}" --tun-peer "${TUN_PEER}"
EOS
chmod 755 rootfs/entrypoint.sh

# 5. package as Docker v1.2 image tar
( cd rootfs && tar --owner=0 --group=0 --numeric-owner --mtime='1970-01-01 00:00:00 UTC' --sort=name -cf ../layer.tar . )
LAYER_SHA=$(sha256sum layer.tar | awk '{print $1}')
MAP_ARCH=$( [ "$ARCH" = "x86_64" ] && echo amd64 || echo arm64 )
cat > config.json <<EOF
{"architecture":"${MAP_ARCH}","os":"linux","config":{"Entrypoint":["/entrypoint.sh"],"Env":["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin","RUST_LOG=info"]},"rootfs":{"type":"layers","diff_ids":["sha256:${LAYER_SHA}"]},"history":[{"created_by":"alpine+iptables+phantun"}]}
EOF
CONFIG_SHA=$(sha256sum config.json | awk '{print $1}')
mv config.json "${CONFIG_SHA}.json"
mkdir "$LAYER_SHA" && mv layer.tar "${LAYER_SHA}/layer.tar" && echo "1.0" > "${LAYER_SHA}/VERSION"
echo "{\"id\":\"${LAYER_SHA}\"}" > "${LAYER_SHA}/json"
cat > manifest.json <<EOF
[{"Config":"${CONFIG_SHA}.json","RepoTags":["phantun-client:0.8.1-alpine"],"Layers":["${LAYER_SHA}/layer.tar"]}]
EOF
echo "{\"phantun-client\":{\"0.8.1-alpine\":\"${LAYER_SHA}\"}}" > repositories
tar -cf phantun-client-alpine.tar manifest.json repositories "${CONFIG_SHA}.json" "${LAYER_SHA}"
# Upload phantun-client-alpine.tar to RouterOS Files (WinBox drag-drop or scp).
```

Result is typically ~10 MB. Host `scratch + busybox` variant (no iptables) is smaller but **will not work on RouterOS** (G7).

### B-R3. Create veth, attach to LAN bridge, add container

Architecture: **one container per VPS**. Attach each veth to the existing LAN bridge (simplest — uses LAN's existing masquerade). Assign each container a free LAN IP (static, outside DHCP pool).

```routeros
# Shared one-time config
/container/config/set tmpdir=disk1/pull
/container/mounts/add name=tun src=/dev/net/tun dst=/dev/net/tun   ;# CRITICAL — missing => container won't start

# Per-VPS: veth + envlist + container
/interface/veth/add name=veth-phantun address=192.168.X.7/24 gateway=192.168.X.1
/interface/bridge/port/add bridge=bridge1 interface=veth-phantun

/container/envs/add list=phantun-env key=VPS_IP     value=<VPS public IP>
/container/envs/add list=phantun-env key=VPS_PORT   value=4567
/container/envs/add list=phantun-env key=LOCAL_PORT value=4567
/container/envs/add list=phantun-env key=TUN_LOCAL  value=192.168.201.2
/container/envs/add list=phantun-env key=TUN_PEER   value=192.168.201.1
/container/envs/add list=phantun-env key=RUST_LOG   value=info

/container/add \
    file=phantun-client-alpine.tar \
    interface=veth-phantun \
    root-dir=disk1/phantun \
    mounts=tun \
    envlist=phantun-env \
    hostname=phantun \
    logging=yes \
    start-on-boot=yes \
    name=phantun
```

### B-R4. **Add the RouterOS route for TUN_PEER subnet (G6)**

This line is the #1 thing users miss. Without it, fake-TCP return packets have nowhere to go.

```routeros
/ip/route/add dst-address=192.168.201.0/24 gateway=192.168.X.7 comment=phantun-return
```

Where `192.168.X.7` is the veth LAN IP of the container, and `192.168.201.0/24` is the subnet containing `TUN_PEER` env var.

### B-R5. Start and verify

```routeros
/container/start phantun
/log/print where topics~"container" and message~"phantun"
```

Expected in the log within ~3 seconds:
```
[entrypoint] RST DROP via nft     (or legacy)
[entrypoint] exec phantun-client --local 0.0.0.0:4567 ...
INFO  client > Remote address is: <VPS IP>:4567
INFO  client > Created TUN device tun-ph
```

Then trigger a WG client connection (Mihomo / RouterOS native WG / whatever):
```
INFO  client   > New UDP client from <WG-client-IP>:<port>
INFO  fake_tcp > Sent SYN to server
INFO  fake_tcp > Connection established     ← this line = success
```

conntrack check on RouterOS:
```routeros
/ip/firewall/connection/print where dst-port=4567 and protocol=tcp
# Expect a row with flags "SAC..." (SEEN-REPLY + ASSURED + CONFIRMED) and state=established.
# If you see state=syn-recv forever → G6 route missing.
# If conntrack has no row at all → firewall dropped the SYN before conntrack.
```

### B-R6. Multi-VPS pattern

To add a second (third, ...) VPS, repeat B-R3 through B-R5 with **different values** for:

| Item | Container 1 | Container 2 | Container N |
|---|---|---|---|
| veth name | `veth-phantun` | `veth-phantun2` | `veth-phantunN` |
| Container LAN IP | `192.168.X.7` | `192.168.X.8` | free IP |
| envlist | `phantun-env` | `phantun2-env` | `phantunN-env` |
| container `name=` | `phantun` | `phantun2` | `phantunN` |
| `root-dir=` | `disk1/phantun` | `disk1/phantun2` | `disk1/phantunN` |
| `TUN_LOCAL` | `192.168.201.2` | **`192.168.202.2`** | `192.168.<N+200>.2` |
| `TUN_PEER` | `192.168.201.1` | **`192.168.202.1`** | `192.168.<N+200>.1` |
| RouterOS route | `192.168.201.0/24 → .X.7` | `192.168.202.0/24 → .X.8` | `192.168.<N+200>.0/24 → ...` |

Why the TUN_LOCAL/TUN_PEER **must differ**: RouterOS routes return packets by destination IP. Same tun-peer across containers → only one gets the return packets. Different tun-peer subnets → each container gets its own return path.

WG subnet (`10.200.0.0/24`) can stay identical across containers — it's scoped per WG tunnel, not a routing concern.

## Phase C — Bridge into your proxy framework

Pick one based on what's installed. The output of all paths is the same: a transparent proxy that forwards selected traffic via WG.

### C-A: daed (no native WG) → sing-box bridge → WG

daed v1.x lacks WG outbound. Use sing-box as a SOCKS5 ⇄ WG converter that daed treats as a normal SOCKS5 node.

Install sing-box (1.11+ for new endpoint format):
```sh
apk add sing-box
```

Write `/etc/sing-box/config.json`:
```json
{
  "log": { "level": "info", "timestamp": true },
  "dns": {
    "servers": [
      {
        "tag": "remote-doh",
        "type": "https",
        "server": "8.8.8.8",
        "server_port": 443,
        "path": "/dns-query",
        "detour": "wg-ep"
      }
    ],
    "final": "remote-doh",
    "strategy": "ipv4_only"
  },
  "inbounds": [
    {
      "type": "socks",
      "tag": "socks-in",
      "listen": "127.0.0.1",
      "listen_port": 1080
    }
  ],
  "endpoints": [
    {
      "type": "wireguard",
      "tag": "wg-ep",
      "address": ["10.200.0.2/32"],
      "private_key": "<client.key from VPS>",
      "mtu": 1300,
      "peers": [
        {
          "address": "127.0.0.1",
          "port": 4567,
          "public_key": "<server.pub from VPS>",
          "allowed_ips": ["0.0.0.0/0", "::/0"],
          "persistent_keepalive_interval": 25
        }
      ]
    }
  ],
  "route": { "final": "wg-ep" }
}
```

```sh
sing-box check -c /etc/sing-box/config.json
uci set sing-box.main.enabled=1 && uci commit sing-box
/etc/init.d/sing-box enable && /etc/init.d/sing-box start
```

The DNS block is **required** if daed runs in `dial_mode: domain*` (default). Without it, sing-box falls back to system resolver → ISP DNS → poisoned for sensitive domains. The DoH query is itself routed through `wg-ep` (`detour: wg-ep`), so it's encrypted end-to-end.

In daed Web UI: add a **SOCKS5 node** pointing to `socks5://127.0.0.1:1080`, place it in a group, route via routing rules.

If daed's main routing has any `dip(8.8.8.8, 8.8.4.4) -> proxy`, the daed-side DNS upstream queries also stay clean.

### C-B: OpenClash / Mihomo — native WG outbound

Mihomo (OpenClash's Meta core) supports WireGuard natively since 2022. Skip sing-box entirely.

Edit OpenClash's active config.yaml (LuCI → OpenClash → Configurations → click your config → Edit), or use the **Overwrite Settings → Custom Override** YAML editor:

```yaml
proxies:
  - name: "wg-vps"
    type: wireguard
    server: 127.0.0.1
    port: 4567
    private-key: "<client.key from VPS>"
    public-key: "<server.pub from VPS>"
    ip: 10.200.0.2
    mtu: 1300
    udp: true
    remote-dns-resolve: true
    dns: ['8.8.8.8', '1.1.1.1']

proxy-groups:
  - name: "PROXY"
    type: select
    proxies:
      - "wg-vps"
      - DIRECT
```

`remote-dns-resolve: true` makes Mihomo resolve through the WG tunnel (DNS sealed inside tunnel — no leak).

Restart OpenClash. Pick the PROXY group's wg-vps in dashboard.

### C-C: sing-box alone (no daed/OpenClash)

Use sing-box's `tun` inbound for transparent proxy. Out of scope for this skill — see sing-box upstream docs.

## Phase D — verification

```sh
# VPS side
ssh root@<vps> 'wg show && journalctl -u phantun-server -n 10 --no-pager'

# OpenWrt side
ssh root@<openwrt> 'pgrep -a phantun-client && ip addr show tun-ph && \
  iptables -nvL OUTPUT | grep 4567 && \
  nft list chain inet fw4 forward | grep tun-ph'

# End-to-end via SOCKS5 (if sing-box bridge in use)
ssh root@<openwrt> 'curl -s -x socks5h://127.0.0.1:1080 https://ipv4.icanhazip.com'
# Expected: returns the VPS public IP, NOT the ISP IP

# Dual-side packet capture during traffic (for diagnostics)
# On VPS: tcpdump -i eth0 -n "tcp port 4567" -c 20
# On OpenWrt: tcpdump -i pppoe-wan -n "host <vps_ip> and tcp port 4567" -c 20
```

WG handshake confirms by `wg show`'s `latest handshake: <seconds> ago` updating within the 25-second keepalive interval.

## Common diagnostic flow when something is broken

1. **VPS phantun-server has no incoming connections (silent log)**: 
   - Run `iptables -t nat -nvL PREROUTING | grep 4567` → if missing, fix G1 (DNAT)
   - `tcpdump -i eth0 -n 'tcp port 4567'` → if SYNs arrive but no SYN-ACK, definitely G1
2. **OpenWrt phantun-client logs "Sent SYN, Connection closed"**:
   - SYN reaches WAN but no SYN-ACK comes back → see step 1 on VPS, OR ISP/middlebox dropping
   - Verify with parallel tcpdump on both sides
3. **WG handshake never completes (`wg show` stays at "(none)" for handshake)**:
   - Phantun link broken OR WG keys mismatched OR sing-box/Mihomo config typo
   - Check sing-box log: `logread -e sing-box | tail -20`, look for `endpoint/wireguard[wg-ep]` errors
4. **DNS resolves to wrong IPs (Facebook range for YouTube CDN)**:
   - Upstream DNS is being intercepted/poisoned. Add DoH detour through tunnel (see C-A DNS block)
   - Or set `remote-dns-resolve: true` in Mihomo proxy config
5. **fw4 reload kills connectivity**:
   - You added `nft insert rule inet fw4 forward ... accept` at runtime instead of UCI zone (G2). Use the UCI zone approach.
6. **RouterOS container: phantun-client logs `Waiting for SYN+ACK timed out` but `/ip/firewall/nat` counter on VPS shows DNAT hits**:
   - Outbound works; return is lost. Check conntrack: `/ip/firewall/connection/print where dst-port=4567`. If state is `syn-recv` and source IP is the TUN_PEER (e.g. `192.168.201.1`) → it's **G6**. Add the route: `/ip/route/add dst-address=192.168.201.0/24 gateway=<container-LAN-IP>`.
7. **RouterOS container image starts and stops within ~1s** with no useful log:
   - Missing `/dev/net/tun` mount. `/container/mounts/add name=tun src=/dev/net/tun dst=/dev/net/tun` and pass `mounts=tun` on `/container/add`.
8. **Mihomo: Hysteria2 via VPS works but WireGuard via same VPS fails for Google/YouTube**:
   - G8. Add `remote-dns-resolve: true` + `dns: ["8.8.8.8","8.8.4.4"]` to the WG proxy block in Mihomo config.
9. **RouterOS: `forward drop invalid` packet counter growing while phantun is retrying**:
   - Could be the return SYN+ACK dropped as invalid (uncommon after G6 is fixed). Either add an explicit accept rule in forward before `drop invalid`:
     ```
     /ip/firewall/filter/add chain=forward action=accept protocol=tcp \
         src-address=<VPS> src-port=<PORT> place-before=<invalid-rule-index>
     ```
     Or more commonly: the drop-invalid hits are unrelated noise. With route G6 + container G7 in place, conntrack transitions to `established` and fasttrack takes over; the explicit accept is typically not required.

## File locations to remember

| What | Where |
|---|---|
| VPS WG config | `/etc/wireguard/wg0.conf`, keys in same dir |
| VPS phantun service | `/etc/systemd/system/phantun-server.service` |
| VPS persisted iptables | `/etc/iptables/rules.v4` (saved by netfilter-persistent) |
| OpenWrt phantun service | `/etc/init.d/phantun-client` |
| OpenWrt firewall UCI | `/etc/config/firewall` (phantun zone) |
| OpenWrt sing-box config | `/etc/sing-box/config.json` |
| OpenClash config | `/etc/openclash/config/<name>.yaml` |
| daed config | SQLite `/etc/daed/wing.db` (edit via Web UI 2023 port, not text file) |
| RouterOS container image tar | Uploaded to `Files/` (WinBox drag-drop) then `/container/add file=...` |
| RouterOS container envs | `/container/envs/print list=phantun-env` |
| RouterOS container mounts | `/container/mounts/print` (must include `src=/dev/net/tun`) |
| RouterOS return-route (G6) | `/ip/route/print where comment~"phantun"` |
| RouterOS per-flow conntrack | `/ip/firewall/connection/print where dst-port=4567` |
| Mihomo config (container) | typically `/root/.config/mihomo/config.yaml` |
| Mihomo runtime API | `http://<mihomo-container-IP>:9090` (proxies, rules, configs, connections endpoints) |

## What this skill does NOT cover

- Multi-peer WG setups (we assume 1 client : 1 server)
- IPv6 inside the tunnel (sing-box's `wg-ep` `address` field can be extended; needs VPS-side v6 NAT)
- Phantun handshake-packet feature (`--handshake-packet`) for stronger TCP fingerprint disguise
- Switching from sing-box bridge to native daed WG (daed roadmap may add this; check current version)
- Building from source (use upstream prebuilt binaries — check signature)

## Quick reference: parameters used in the canonical example

| Param | Value | Notes |
|---|---|---|
| Phantun TCP port | 4567 | Change to 443/8443 for stronger HTTPS disguise |
| WG UDP port (VPS local only) | 51820 | Bound to `127.0.0.1`, never exposed |
| WG subnet | 10.200.0.0/24 | server `.1`, client `.2` |
| MTU | 1300 | Conservative; prevents PMTU black holes on PPPoE |
| VPS phantun TUN | tun0, peer 192.168.200.2 | `--tun-peer` is the DNAT target (G1) |
| OpenWrt phantun TUN | tun-ph, peer 192.168.201.1 | Distinct subnet from VPS side |
| sing-box SOCKS5 | 127.0.0.1:1080 | Listen local only |

---
> Source: [lilei9634/phantun-wireguard-tunnel-skill](https://github.com/lilei9634/phantun-wireguard-tunnel-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
