## embewi-core

> Contrôleur Kubernetes qui pilote des devices ESP32 via le contrat

# CLAUDE.md — Embewi Core (contrôleur Kubernetes)

Contrôleur Kubernetes qui pilote des devices ESP32 via le contrat
[`embewi`](https://github.com/iobewi/embewi) (`v1alpha1`). Les devices
(firmware [`embewi-agent-esp`](https://github.com/iobewi/embewi-agent-esp))
sont joignables en HTTPS :443 sur leur IP Wi-Fi, exposés comme endpoints d'un
Service Kubernetes selectorless.

Langue du projet : **français** (commentaires, doc, messages). Garder cette langue.

## Contrat de référence

Spec normative : **`contract/docs/embewi-contract-v2.md`** (protocole `v1alpha1`).
C'est la **source de vérité** pour toute interaction Core ↔ Agent.
`git submodule update --init` si `contract/` est vide.

Les sections **[NORMATIF]** sont des contraintes d'implémentation.
Les sections **[RÉSERVE]** sont hors MVP — ne pas implémenter sans décision.

## CRDs à implémenter

### `McuNode` — device physique enrôlé

```yaml
apiVersion: embewi.io/v1alpha1
kind: McuNode
metadata:
  name: embewi-a1b2c3
spec:
  nodeId: embewi-a1b2c3
  tokenRef:                        # référence au Secret K8s portant le token Bearer
    name: embewi-a1b2c3-token
    namespace: embewi
  tlsSecretRef:                    # optionnel — Secret kubernetes.io/tls (compat cert-manager)
    name: embewi-a1b2c3-tls        # absent = agent garde son cert auto-signé de build
status:
  ip: "192.168.10.42"             # mis à jour depuis heartbeat.ip à chaque réception
  lastHeartbeat: "2026-06-29T10:00:03Z"
  state: running
  apiVersion: v1alpha1             # négocié depuis GET /info (api_versions), absent → v1alpha1 supposé
  tlsCertDigest: "a3f9..."         # sha256(tls.crt+tls.key) du dernier cert TLS appliqué (suivi Core-side)
  lastRebootRequested: ""          # miroir de l'annotation embewi.io/reboot-requested traitée
  conditions:
    - type: Provisioned
      status: "True"
      reason: ProvisioningComplete
    - type: Ready
      status: "True"
      reason: HeartbeatOK
      message: "Heartbeat reçu il y a 3s"
```

Conditions McuNode (§8a) :

| Condition | `reason` | `status` | Déclencheur |
|---|---|---|---|
| `Provisioned` | `ProvisioningComplete` | True | node_id + token établis |
| `Provisioned` | `ProvisioningPending` | False | premier boot, portail AP actif |
| `Ready` | `HeartbeatOK` | True | heartbeat reçu < 2× période (10 s) |
| `Ready` | `HeartbeatTimeout` | False | aucun heartbeat > seuil |
| `Ready` | `NotProvisioned` | Unknown | jamais enrôlé |

### `McuDeployment` — déploiement firmware + workload

```yaml
apiVersion: embewi.io/v1alpha1
kind: McuDeployment
metadata:
  name: wheel-left
spec:
  nodeName: embewi-a1b2c3          # pin explicite privilégié
  firmware: registry.local/embewi/wheel-controller:v1.1.0
  configMapRef: wheel-left-gpio    # optionnel — absent = défauts build
  appPort: 9090                    # optionnel — 0/absent = pas de reconfiguration (POST /app/port)
status:
  deploymentId: wheel-controller-1.1.0
  activeSlot: ota_0
  firmwareDigest: "sha256:..."
  conditions:
    - type: Progressing
      status: "False"
      reason: DeploymentComplete
    - type: Available
      status: "True"
      reason: WorkloadReady
```

Conditions McuDeployment (§8a) :

| Condition | `reason` | `status` | Déclencheur |
|---|---|---|---|
| `Progressing` | `OTAInProgress` | True | PUT /ota/write ou pending_verify |
| `Progressing` | `DeploymentComplete` | False | OTA terminé, firmware stable |
| `Progressing` | `OTAFailed` | False | rollback ou état failed |
| `Available` | `WorkloadReady` | True | EndpointSlice.ready=true |
| `Available` | `PendingVerification` | False | state=pending_verify |
| `Available` | `DeviceDegraded` | False | state ∈ {degraded, rollback, failed} |
| `Available` | `HeartbeatTimeout` | False | device plus joignable |

**`Ready` synthétique** : `Ready=True` ← `Progressing=False` ET `Available=True`.
Compatible `kubectl wait mcudeployment/wheel-left --for=condition=Available`.

### `McuConfigMap` — config runtime

```yaml
apiVersion: embewi.io/v1alpha1
kind: McuConfigMap
metadata:
  name: wheel-left-gpio
data:
  gpio_button: "9"
  gpio_ws2812: "48"
  ntp_server: "ntp.local"
  # clés arbitraires — opaques pour l'agent, sémantique côté app
```

Limites NVS agent (à valider côté Core avant push) :
- Clé : 15 caractères max
- Valeur : 63 caractères max

Clés réservées agent (préfixe `_`) : ignorées silencieusement par `POST /config`.

## API agent à appeler (Core → ESP, contrat §4)

Préfixe : `/v1alpha1`. HTTPS :443. `Authorization: Bearer <token>` sur **tous** les endpoints.

### Séquence de réconciliation complète

```text
1. GET /info          → lit staged.state (idempotence), config_generation, app_port
2. GET /health        → optionnel, confirme l'état local avant OTA
3. POST /config       → si McuConfigMap diverge du NVS courant
4. POST /ota/prepare  → annonce le firmware (chip, size, digest, deployment_id)
5. PUT  /ota/write    → stream du .bin par chunks (Content-Range)
6. POST /ota/activate → set_boot_partition + reboot
7. [heartbeat] state=pending_verify → self-check en cours
8. [heartbeat] state=running + ota_validated=true → déploiement confirmé
```

Ordre canonique si config + OTA dans la même réconciliation :
**POST /config d'abord, OTA ensuite** — un seul reboot couvre les deux.

### Idempotence via `staged.state` (§6)

| `staged.state` | Action Core |
|---|---|
| `none` | Repartir de `/ota/prepare` |
| `written` + bon digest | Sauter le write, aller à `/ota/activate` |
| `written` + digest périmé | Re-préparer + ré-écrire |
| `activating` | Attendre le heartbeat (timeout négatif §3) |

Le `deployment_id` est persisté dans `staged` dès `/ota/write`
(header `X-Embewi-Deployment-Id`) — `GET /info` l'expose avant même l'activate.

### Write reprenable — Content-Range (§4)

```text
Content-Range: bytes <start>-<end>/<total>   (bornes inclusives)
```

- `start=0` ou header absent → nouvelle session
- `start>0` aligné sur `written` → reprise (CONTINUE)
- `start>0` désaligné → `416 {"error":"range_mismatch","written":N}` → resync

Le handle OTA et le SHA-256 incrémental survivent aux déconnexions TCP côté agent.

### Codes d'erreur stables (§4b) — mapper en Events K8s

**`POST /ota/prepare`** (`reason`, HTTP 200, `accepted:false`) :

| `reason` | Event Core |
|---|---|
| `chip_mismatch` | `OTARejectedChip` |
| `layout_mismatch` | `OTARejectedLayout` |
| `idf_incompatible` | `OTARejectedIdf` |
| `size_too_large` | `OTARejectedSize` |
| `busy` | `OTABusy` |

**`PUT /ota/write`** (`status`, HTTP 200 sauf mention) :

| `status` / `error` | Event Core |
|---|---|
| `written` | `OTAWritten` |
| `digest_mismatch` | `OTADigestMismatch` |
| `write_failed` | `OTAWriteFailed` |
| `ota_begin_failed` (500) | `OTABeginFailed` |
| `range_mismatch` (416) | — (resync, attendu) |

**Règle** : un refus métier légitime répond HTTP 200 avec le code dans le corps.
Les `4xx/5xx` sont des erreurs de protocole. Ne pas inverser.

### Rotation de token (§4) — sans coupure, via `previousToken`

```text
1. Écriture ATOMIQUE (un seul kubectl patch) : Secret["token"]=newToken +
   Secret["previousToken"]=oldToken. Jamais newToken seul — sans
   previousToken, un crash Core juste après l'écriture rend le device
   injoignable en inbound (seul recours : portail captif).
2. reconcileTokenRotation (McuNodeReconciler) détecte previousToken → rejoue
   POST /token (Bearer previousToken, corps newToken) à chaque reconcile
   tant que non confirmé.
3. Device commite newToken en NVS avant de répondre 200 (atomique, §4) — seul
   newToken valide dès la réponse.
4. Premier heartbeat authentifié en newToken → previousToken effacé du
   Secret par le Core (clearPreviousToken, internal/heartbeat/server.go).
```

Le heartbeat handler accepte `token` ET `previousToken` pendant la fenêtre de
rotation (`validateToken`) — ce n'est pas une anomalie tant que le device n'a
pas confirmé le nouveau token.

### `POST /app/port` — port applicatif (§4, sur `McuDeployment`)

`spec.appPort` (McuDeployment) comparé à `status.appPort` (McuNode, peuplé
depuis `GET /info`) → push si divergent. **Pas de reboot** : l'agent
redémarre son service app à chaud. `status.appPort` mis à jour immédiatement
après un push confirmé (rien d'autre ne le rafraîchit en phase Deployed).

### `POST /tls/cert` — rotation de certificat sans cycle OTA (§4, sur `McuNode`)

`spec.tlsSecretRef` référence un Secret `kubernetes.io/tls` (`tls.crt`/
`tls.key` — compatible cert-manager, renouvellement automatique
transparent). Suivi par `sha256(cert+key)` dans `status.tlsCertDigest`
(Core-side : contrairement à la config §4a, l'agent n'expose aucun digest de
cert via `GET /info`). Différent → `POST /tls/cert` puis **reboot requis**
(effectif au prochain `embewi_http_start()`) ; le digest n'est marqué à jour
qu'après un reboot confirmé.

### Reboot à la demande — annotation (pas dans le contrat, Core seul)

Pas d'équivalent `kubectl rollout restart` pour un CRD (verbe spécifique aux
`deployment`/`daemonset`/`statefulset`) :

```bash
kubectl annotate mcunode <name> embewi.io/reboot-requested="$(date +%s)" --overwrite
```

Valeur comparée à `status.lastRebootRequested` (nonce opaque) → différente
→ `POST /reboot`. Pas de nettoyage d'annotation requis. **Découplé de
`delete McuNode`** délibérément : le CRD représente le device, il ne le
possède pas (même logique que `kubectl delete node` qui ne touche jamais à
la VM sous-jacente) — supprimer l'objet ne déclenche aucune action physique.

## Flux sortants reçus (ESP → Core, contrat §5)

### Heartbeat — HTTPS POST toutes les 5 s

```json
{
  "node_id": "embewi-a1b2c3",
  "ip": "192.168.10.42",
  "ts": 1710000000,
  "state": "running",
  "deployment_id": "wheel-controller-1.1.0",
  "firmware_digest": "sha256:...",
  "ota_validated": true,
  "uptime_ms": 120034,
  "heap_free": 82344,
  "rssi": -61,
  "config_generation": 2,
  "temp_celsius": 41.2,
  "task_hwm_min": 1536
}
```

Champs **requis** : `node_id`, `ip`, `ts`, `state`, `ota_validated`, `config_generation`.

**`ip` est la source de vérité pour `EndpointSlice.endpoints[].addresses`** (§8) :
mettre à jour à chaque heartbeat reçu — l'IP source TCP n'est pas fiable.

`ota_validated` = `true` uniquement après `mark_valid` côté bootloader.
Un device en `pending_verify` émet `ota_validated: false` — ne jamais marquer
`Available=True` avant `ota_validated=true` + `state=running` + bon `deployment_id`.

`temp_celsius` = `-127.0` si capteur SoC indisponible — filtrer cette valeur.

`ts` ≈ 0 si NTP pas encore synchronisé (boot récent) — ne pas alerter sur `ts` seul.

### Logs — WebSocket wss (streaming)

L'agent ouvre une connexion WS **cliente** (outbound) vers
`wss://<ctrl_url_host>:<port>/v1alpha1/logs`.

Format par frame :
```json
{
  "ts": 1719392051,
  "node": "embewi-a1b2c3",
  "workload": "wheel-controller",
  "level": "raw",
  "msg": "I (10352) embewi.ota: write OK 983040 octets slot=ota_1"
}
```

Garantie : best-effort, sans replay inter-reconnexion. Les événements
OTA/lifecycle critiques passent aussi par HTTPS POST (`/v1alpha1/logs`).

## Effets Kubernetes à piloter (§8/§8a/§8b)

### EndpointSlice — routage vers le workload

```yaml
endpoints:
  - addresses: ["192.168.10.42"]   # ← heartbeat.ip, mis à jour à chaque réception
    conditions:
      ready: true                  # piloté par ota_validated + state, jamais statique
ports:
  - port: 8080                     # ← app_port (GET /info)
```

Pilotage de `ready` :

```text
heartbeat OK + state=running + ota_validated=true  → ready=true
state=pending_verify                               → ready=false
heartbeat perdu (> seuil)                          → ready=false
state ∈ {degraded, rollback, failed}               → ready=false
```

### Pipeline métriques Prometheus (§8b)

Le Core expose `/metrics` sur un port dédié (ex. `:9090`).
Chaque heartbeat met à jour les gauges du device via le label `node_id`.

Labels communs : `node_id`, `workload`, `chip`.

| Métrique | Type | Source |
|---|---|---|
| `mcunode_heap_free_bytes` | gauge | `heap_free` |
| `mcunode_wifi_rssi_dbm` | gauge | `rssi` |
| `mcunode_uptime_seconds` | gauge | `uptime_ms / 1000` |
| `mcunode_temperature_celsius` | gauge | `temp_celsius` (filtrer `-127.0`) |
| `mcunode_task_stack_hwm_bytes` | gauge | `task_hwm_min` |
| `mcunode_last_heartbeat_timestamp` | gauge | `ts` |
| `mcunode_config_generation` | gauge | `config_generation` |
| `mcunode_ota_validated` | gauge | `ota_validated` → 0/1 |

Noms stables — ne pas renommer sans réviser les dashboards.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: embewi-core
  namespace: embewi
spec:
  selector:
    matchLabels:
      app: embewi-core
  endpoints:
    - port: metrics
      path: /metrics
      interval: 30s
```

## Machine d'état agent (§2)

```text
booting → running          (image déjà validée)
booting → pending_verify   (boot post-activate)
pending_verify → running   (self-check OK, mark_valid — 15 s max)
pending_verify → rollback  (self-check KO ou deadline)
running → degraded         (check KO en cours de route)
rollback → running         (reboot sur image précédente)
* → failed                 (rollback impossible)
```

**`FAILED` : le device ne reboot pas.** Il reste joignable — repousser une image
saine via OTA. Ne jamais interrompre le heartbeat côté Core sur un device `failed`.

**`pending_verify` : heartbeat émis en continu** (pas de silence — le silence est
indistinguable d'un crash). Le Core attend la confirmation (`state=running` +
`ota_validated=true` + bon `deployment_id`) avec un timeout négatif.

## Sécurité (§1)

- Token Bearer par node, comparaison **temps constant** côté agent.
- Transport : HTTPS obligatoire (le scheme `http://` dans `ctrl_url` est forcé
  en `https://` par l'agent — c'est une erreur de config, pas un fallback).
- Profil prod agent : Secure Boot v2 + Flash Encryption + `CONFIG_EMBEWI_VERIFY_CORE_CERT`
  (CA du Core embarquée dans le binaire, valide les flux sortants).
- Filtrage IP inbound agent (`CONFIG_EMBEWI_ENABLE_IP_FILTER`) :
  rejette les connexions hors `allowed_cidr` avant tout handler.
  Pousser via McuConfigMap : `{"data":{"allowed_cidr":"10.42.0.0/16"}}`.
  Recommandation : CIDR du cluster K8s, pas l'IP exacte du Pod Core.

## Politique de binding (§7)

```text
1 McuDeployment → résout EXACTEMENT 1 McuNode.
0 match     → NoDeviceMatched
>1 match    → AmbiguousBinding
node occupé → DeviceBusy   (first-bound wins)
```

Privilégier `spec.nodeName` (pin explicite) plutôt que `spec.nodeSelector`.

## Hors MVP — [RÉSERVE]

Ne pas implémenter sans décision explicite :

```text
- OTA pull (device-initiated) pour devices NATés
- Reprise OTA inter-reboot (coupure + redémarrage mid-transfer)
- Pod IP logique + dataplane proxy/NAT
- McuDeploymentSet (fleet rolling/canary) — génère des McuDeployment unitaires
- ValidatingAdmissionWebhook McuConfigMap (validation sémantique des valeurs)
- DELETE /config/{key} (reset clé individuelle)
- Hot-reload config sans reboot
- Virtual Kubelet provider (ESP comme vrais Nodes K8s)
- Double-token overlap (rotation sans fenêtre de coupure)
- McuConfigMap versionné (rollback config indépendant du rollback firmware)
```

## Référence rapide

| Ressource | Rôle |
|---|---|
| `contract/docs/embewi-contract-v2.md` | Spec normative complète |
| `contract/docs/liaison/` | Matrice de traçabilité contrat↔code + suivi des issues cross-repo |
| `embewi-agent-esp` | Firmware device (source, build, tests) |
| `docs/` (agent) | Doc Sphinx : architecture, API, config, sécurité, workload SDK |
| `docs/core/controllers.md` | Détail des reconcilers Core (phases OTA, app_port, TLS cert, reboot) |
| `docs/usage/operations.md` | Procédures opérationnelles (rotation token, reboot, métriques) |

---
> Source: [iobewi/embewi-core](https://github.com/iobewi/embewi-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
