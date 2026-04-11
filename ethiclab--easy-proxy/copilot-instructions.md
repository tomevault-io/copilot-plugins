## easy-proxy

> > Guida per agenti AI (Claude, Copilot, Codex, ChatGPT, etc.)

# easy-proxy — AGENTS.md

> Guida per agenti AI (Claude, Copilot, Codex, ChatGPT, etc.)
> Aggiornato: 2026-04-02

---

## Contesto del progetto

**easy-proxy** è una CLI bash+Docker che gestisce nginx come reverse proxy locale con SSL Let's Encrypt automatico via DNS-01. Sviluppato da EthicLab per:
- Local dev SSL (OAuth/Cognito testing con HTTPS reale)
- Demo environment professionale con domini reali

Versione attuale: **2.0.0** — UC1 implementato (IONOS DNS automation).

---

## Come orientarsi velocemente

```
easy-proxy/
├── easy                       # Entry point — NON modificare
├── commands/proxy.sh          # Tutti i comandi — modificare qui
├── easyhome/
│   ├── skeleton.js            # Template renderer — Node.js, zero deps
│   ├── nginx.conf             # Config nginx main
│   ├── templates/*.conf       # Template vhost
│   ├── add_subdomain_http     # Genera vhost HTTP
│   ├── add_subdomain_https    # Genera vhost HTTPS
│   ├── ionos-config-helper.sh # Helper credenziali IONOS
│   └── easy-proxy-start       # Entrypoint container
├── Dockerfile                 # Container easy-proxy (usa base v2.0)
├── Dockerfile.build           # Build base image nginx-certbot:2.0
├── CLAUDE.md                  # Guida per Claude Code
├── STATE.md                   # WIP corrente
├── UC1_LOCAL_SSL_SETUP.md     # Quick start UC1
└── USE_CASES.md               # Use case completi
```

---

## Stato attuale (2026-04-02)

### Già fatto ✅
- Base image `ethiclab/nginx-certbot:2.0` costruita (Alpine + nginx 1.26 + certbot-dns-ionos + Node 20)
- `easy proxy certbot-ionos <domain>` implementato — legge credenziali da `pass` o env vars
- `skeleton.py` (Python 2) rimpiazzato da `skeleton.js` (Node.js, zero dipendenze)
- Tutti i test locali passano

### Da fare 🔴
1. **Push Docker Hub**: `docker push ethiclab/nginx-certbot:2.0`
2. **Test reale IONOS**: serve `pass insert ionos/api-key` + `pass insert ionos/api-secret`
3. **Split-view DNS** (Phase 2): dnsmasq locale per `*.dev.ethiclab.it → 127.0.0.1`
4. **Register.it support** (Phase 3): plugin certbot-dns-register-it

---

## Regole di sviluppo

### Non toccare
- `easy` (entry point) — solo aggiungere comandi in `commands/proxy.sh`
- Non aggiungere `-it` ai `docker exec` non-interattivi (rompe uso in script)
- Non cambiare `user nginx;` in `nginx.conf` — Alpine usa `nginx`, non `www-data`

### Aggiungere un nuovo DNS provider

1. Aggiungere plugin nel `Dockerfile.build`:
   ```dockerfile
   pip install certbot-dns-<provider>
   ```

2. Aggiungere subcommand in `commands/proxy.sh`:
   ```bash
   if [[ "certbot-<provider>" == "$2" ]]; then
     # ... stesso pattern di certbot-ionos
   fi
   ```

3. Aggiungere helper credentials in `easyhome/<provider>-config-helper.sh`

4. Aggiornare help in `__easy_command_proxy_help`

### Modificare template nginx

I template in `easyhome/templates/` usano variabili bash-style:
- `$server_name`, `$domain`, `$location_path`, `$location_target` → sostituite da `skeleton.js`
- `\$upstream`, `\$host` etc. → escape Cheetah, vengono unescapati a `$upstream` nell'output

### Testare senza credenziali IONOS

```bash
export EASY_DIR="$HOME/ethiclab/lab/easy-proxy"
export EASY_LETSENCRYPT_DIR="$HOME/.easy-proxy/letsencrypt"
export EASY_DOMAINS_DIR="$HOME/.easy-proxy/domains"
export EASY_LETSENCRYPT_EMAIL="test@ethiclab.it"
export PATH="$EASY_DIR:$PATH"
mkdir -p "$EASY_LETSENCRYPT_DIR" "$EASY_DOMAINS_DIR"

easy proxy create
easy proxy new http test.ethiclab.it ethiclab.it http://host.docker.internal:8010
easy proxy reload
curl -s -o /dev/null -w "%{http_code}" http://localhost/   # 404 = OK
easy proxy destroy
```

---

## Architettura del container

```
ethiclab/nginx-easy  (FROM ethiclab/nginx-certbot:2.0)
 │
 ├─ nginx 1.26.3 (user: nginx, config: /usr/local/share/easy/nginx.conf)
 │   └─ include /domains/*/*.conf  ← vhost generati da skeleton.js
 │
 ├─ certbot 5.4.0
 │   └─ plugins: dns-ionos, dns-route53, dns-cloudflare, dns-digitalocean
 │
 ├─ Node.js 20 (per skeleton.js)
 │
 └─ Volumes:
     /domains             ← EASY_DOMAINS_DIR (host)
     /etc/letsencrypt     ← EASY_LETSENCRYPT_DIR (host)
     /usr/local/share/easy ← easyhome/ (repo)
```

---

## Handoff precedente

Vedi `.agents/008-easy-proxy-uc1.md` per il contesto della sessione 2026-04-02.

---

## Referenze

- IONOS API: https://developer.hosting.ionos.it/
- certbot-dns-ionos: https://github.com/helgeholzmann/certbot-dns-ionos
- nginx Alpine docs: https://wiki.alpinelinux.org/wiki/Nginx

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ethiclab)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ethiclab)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
