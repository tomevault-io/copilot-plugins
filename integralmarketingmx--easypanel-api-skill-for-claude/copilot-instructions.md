## easypanel-api-skill-for-claude

> Manage EasyPanel server - deploy apps, manage projects, services, databases, domains, ports, mounts, backups, monitoring and infrastructure via tRPC API. All endpoints validated 2026-04-06.


# EasyPanel API Skill (v2 - Validated)

**Purpose**: Interact with EasyPanel server to manage projects, apps, databases, domains, ports, mounts, backups, monitoring, and deployments via tRPC API.

> All endpoints below were **tested and validated** against a live EasyPanel instance. Invalid procedures from v1 have been corrected.

## Connection Details

- **URL**: `https://<YOUR_EASYPANEL_HOST>:3000`
- **API Base**: `https://<YOUR_EASYPANEL_HOST>:3000/api/trpc`
- **Auth**: Bearer token from EasyPanel > Settings > Users > API Token
- **Use `-k` flag** if your instance uses self-signed certs

## Authentication

All requests use Bearer token in header:
```bash
-H "Authorization: Bearer <TOKEN>"
-H "Content-Type: application/json"
```

Generate a token in EasyPanel UI: Settings > Users > Generate API Token.
Or via API: `users.generateApiToken` (see Users section below).

## API Pattern

EasyPanel uses **tRPC**. All calls follow this pattern:

### Query (GET - read operations)
```bash
curl -sk "<API_BASE>/<procedure>?input=<URL_ENCODED_JSON>" \
  -H "Authorization: Bearer <TOKEN>"
```

Input must be URL-encoded: `{"json":{"key":"value"}}` -> `%7B%22json%22%3A%7B%22key%22%3A%22value%22%7D%7D`

### Mutation (POST - write operations)
```bash
curl -sk "<API_BASE>/<procedure>" \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"json": { ...params }}'
```

### Response Pattern
- Success: `{"result":{"data":{"json": <data>}}}`
- Success (void): `{"result":{"data":{"json": null}}}`
- Error: `{"error":{"json":{"message":"...","code":-32600,"data":{"zodErrors":{...}}}}}`
- Zod errors reveal required fields and types — use them to discover missing parameters

---

## PROJECTS

| Action | Procedure | Method | Validated Input |
|--------|-----------|--------|-----------------|
| List projects | `projects.listProjects` | GET | none |
| List projects + services | `projects.listProjectsAndServices` | GET | none |
| Can create project | `projects.canCreateProject` | GET | none |
| Create project | `projects.createProject` | POST | `{"json":{"name":"my-project"}}` |
| Inspect project | `projects.inspectProject` | GET | `{"json":{"projectName":"my-project"}}` |
| Update project env | `projects.updateProjectEnv` | POST | `{"json":{"projectName":"x","env":"KEY=value"}}` |
| Get docker containers | `projects.getDockerContainers` | GET | `{"json":{"projectName":"x","service":"svc-name"}}` |
| Destroy project | `projects.destroyProject` | POST | `{"json":{"name":"my-project"}}` |

> **Note**: `destroyProject` uses `name`, NOT `projectName`

---

## APP SERVICES (services.app.*)

### Lifecycle
| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| Create app | `services.app.createService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Deploy | `services.app.deployService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Start | `services.app.startService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Stop | `services.app.stopService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Restart | `services.app.restartService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Destroy | `services.app.destroyService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Inspect | `services.app.inspectService` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Get exposed ports | `services.app.getExposedPorts` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Refresh deploy token | `services.app.refreshDeployToken` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |

### Source Configuration
| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| Set image source | `services.app.updateSourceImage` | POST | `{"json":{"projectName":"x","serviceName":"y","image":"nginx:alpine","username":"","password":""}}` |
| Set GitHub source | `services.app.updateSourceGithub` | POST | `{"json":{"projectName":"x","serviceName":"y","owner":"org","repo":"repo","ref":"main","path":"/","autoDeploy":false}}` |
| Set Git source | `services.app.updateSourceGit` | POST | `{"json":{"projectName":"x","serviceName":"y","repo":"https://...","ref":"main","path":"/"}}` |
| Set Dockerfile source | `services.app.updateSourceDockerfile` | POST | `{"json":{"projectName":"x","serviceName":"y","dockerfile":"FROM nginx:alpine"}}` |
| Enable GitHub deploy | `services.app.enableGithubDeploy` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` (source must be GitHub) |
| Disable GitHub deploy | `services.app.disableGithubDeploy` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` (source must be GitHub) |

### App Settings
| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| Update env vars | `services.app.updateEnv` | POST | `{"json":{"projectName":"x","serviceName":"y","env":"KEY=val\nKEY2=val2"}}` |
| Update deploy config | `services.app.updateDeploy` | POST | `{"json":{"projectName":"x","serviceName":"y","replicas":1,"command":"","zeroDowntime":true}}` |
| Update resources | `services.app.updateResources` | POST | `{"json":{"projectName":"x","serviceName":"y","memoryLimit":256,"memoryReservation":128,"cpuLimit":0.5,"cpuReservation":0.25}}` |
| Update build | `services.app.updateBuild` | POST | `{"json":{"projectName":"x","serviceName":"y","buildArgs":""}}` (requires valid source) |
| Update maintenance | `services.app.updateMaintenance` | POST | `{"json":{"projectName":"x","serviceName":"y","enabled":false}}` |
| Update basic auth | `services.app.updateBasicAuth` | POST | `{"json":{"projectName":"x","serviceName":"y","enabled":false,"credentials":[]}}` |
| Update redirects | `services.app.updateRedirects` | POST | `{"json":{"projectName":"x","serviceName":"y","redirects":[]}}` |
| Update scripts | `services.app.updateScripts` | POST | `{"json":{"projectName":"x","serviceName":"y","scripts":[]}}` (scripts is array, NOT object) |

---

## DATABASES

### Create Database Services
| Type | Procedure | Method | Input |
|------|-----------|--------|-------|
| PostgreSQL | `services.postgres.createService` | POST | `{"json":{"projectName":"x","serviceName":"db","password":"pass"}}` |
| MySQL | `services.mysql.createService` | POST | `{"json":{"projectName":"x","serviceName":"db","password":"pass","rootPassword":"root"}}` |
| MariaDB | `services.mariadb.createService` | POST | `{"json":{"projectName":"x","serviceName":"db","password":"pass","rootPassword":"root"}}` |
| Redis | `services.redis.createService` | POST | `{"json":{"projectName":"x","serviceName":"cache","password":"pass"}}` |
| MongoDB | `services.mongo.createService` | POST | `{"json":{"projectName":"x","serviceName":"db","password":"pass"}}` |

### Database Operations (per type: postgres/mysql/mariadb/redis/mongo)
| Action | Procedure Pattern | Method |
|--------|-------------------|--------|
| Inspect | `services.<type>.inspectService` | GET |
| Destroy | `services.<type>.destroyService` | POST |
| Enable | `services.<type>.enableService` | POST |
| Disable | `services.<type>.disableService` | POST |
| Expose port | `services.<type>.exposeService` | POST |
| Update credentials | `services.<type>.updateCredentials` | POST |
| Update resources | `services.<type>.updateResources` | POST |
| Update advanced | `services.<type>.updateAdvanced` | POST |
| Enable DbGate | `services.<type>.enableDbGate` | POST |
| Disable DbGate | `services.<type>.disableDbGate` | POST |

PostgreSQL-specific: `services.postgres.enablePgWeb` / `disablePgWeb`
MySQL/MariaDB-specific: `services.mysql.enablePhpMyAdmin` / `disablePhpMyAdmin`
Redis-specific: `services.redis.enableRedisCommander` / `disableRedisCommander`
MongoDB-specific: `services.mongo.enableMongoExpress` / `disableMongoExpress`

All use: `{"json":{"projectName":"x","serviceName":"y"}}`

---

## DOMAINS (domains.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List domains | `domains.listDomains` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Get primary domain | `domains.getPrimaryDomain` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Set primary domain | `domains.setPrimaryDomain` | POST | `{"json":{"projectName":"x","serviceName":"y","domainId":"..."}}` |
| Create domain | `domains.createDomain` | POST | see below |
| Update domain | `domains.updateDomain` | POST | similar to create |
| Delete domain | `domains.deleteDomain` | POST | `{"json":{"projectName":"x","serviceName":"y","id":"domain-id"}}` |

### domains.createDomain Full Schema
```json
{
  "json": {
    "projectName": "my-project",
    "serviceName": "my-app",
    "id": "unique-domain-id",
    "host": "app.example.com",
    "https": true,
    "path": "/",
    "middlewares": [],
    "certificateResolver": "letsencrypt",
    "wildcard": false,
    "destinationType": "service",
    "serviceDestination": {
      "port": 3000,
      "protocol": "http",
      "projectName": "my-project",
      "serviceName": "my-app"
    }
  }
}
```

- `certificateResolver`: `"letsencrypt"` or `"none"`
- `destinationType`: `"service"` or `"custom"`
- `serviceDestination.protocol`: `"http"` or `"https"`

---

## PORTS (ports.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List ports | `ports.listPorts` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Create port | `ports.createPort` | POST | `{"json":{"projectName":"x","serviceName":"y","values":{"published":8080,"target":80,"protocol":"tcp"}}}` |
| Update port | `ports.updatePort` | POST | `{"json":{"projectName":"x","serviceName":"y","index":0,"values":{...}}}` |
| Delete port | `ports.deletePort` | POST | `{"json":{"projectName":"x","serviceName":"y","index":0}}` |
| Delete all ports | `ports.deleteAllPorts` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |

---

## MOUNTS (mounts.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List mounts | `mounts.listMounts` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Create mount | `mounts.createMount` | POST | `{"json":{"projectName":"x","serviceName":"y","values":{...}}}` |
| Update mount | `mounts.updateMount` | POST | `{"json":{"projectName":"x","serviceName":"y","index":0,"values":{...}}}` |
| Delete mount | `mounts.deleteMount` | POST | `{"json":{"projectName":"x","serviceName":"y","index":0}}` |

---

## MONITORING (monitor.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| System stats | `monitor.getSystemStats` | GET | none |
| Advanced stats (CPU/mem/net history) | `monitor.getAdvancedStats` | GET | none |
| Storage stats per service | `monitor.getStorageStats` | GET | none |
| Docker task stats | `monitor.getDockerTaskStats` | GET | none |
| Monitor table (per container) | `monitor.getMonitorTableData` | GET | none |
| Service stats | `monitor.getServiceStats` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |

---

## ACTIONS (actions.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List actions | `actions.listActions` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Get action | `actions.getAction` | GET | `{"json":{"actionId":"..."}}` |
| Kill action | `actions.killAction` | POST | `{"json":{"actionId":"..."}}` |

---

## SETTINGS (settings.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| Get server IP | `settings.getServerIp` | GET | none |
| Get panel domain | `settings.getPanelDomain` | GET | none |
| Set panel domain | `settings.setPanelDomain` | POST | `{"json":{...}}` |
| Get service domain | `settings.getServiceDomain` | GET | none |
| Set service domain | `settings.setServiceDomain` | POST | `{"json":{...}}` |
| Get GitHub token | `settings.getGithubToken` | GET | none |
| Set GitHub token | `settings.setGithubToken` | POST | `{"json":{"token":"ghp_..."}}` |
| Get Let's Encrypt email | `settings.getLetsEncryptEmail` | GET | none |
| Set Let's Encrypt email | `settings.setLetsEncryptEmail` | POST | `{"json":{"email":"..."}}` |
| Get Docker version | `settings.getDockerVersion` | GET | none |
| Check for updates | `settings.checkForUpdates` | POST | none |
| Check Docker update | `settings.checkDockerUpdate` | POST | none |
| Restart EasyPanel | `settings.restartEasypanel` | POST | none |
| System prune | `settings.systemPrune` | POST | none |
| Cleanup Docker images | `settings.cleanupDockerImages` | POST | none |
| Cleanup Docker builder | `settings.cleanupDockerBuilder` | POST | none |
| Get daily cleanup config | `settings.getDailyDockerCleanup` | GET | none |
| Set daily cleanup config | `settings.setDailyDockerCleanup` | POST | `{"json":{...}}` |
| Change credentials | `settings.changeCredentials` | POST | `{"json":{...}}` |
| Refresh server IP | `settings.refreshServerIp` | POST | none |
| Get demo mode | `settings.getDemoMode` | GET | none |
| Get Google Analytics ID | `settings.getGoogleAnalyticsMeasurementId` | GET | none |

---

## USERS (users.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List users | `users.listUsers` | GET | none |
| Create user | `users.createUser` | POST | `{"json":{"email":"...","password":"...","admin":false}}` |
| Update user | `users.updateUser` | POST | `{"json":{...}}` |
| Destroy user | `users.destroyUser` | POST | `{"json":{"userId":"..."}}` |
| Generate API token | `users.generateApiToken` | POST | `{"json":{"userId":"..."}}` |
| Revoke API token | `users.revokeApiToken` | POST | `{"json":{"userId":"..."}}` |

---

## COMMON SERVICE OPS (services.common.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| Rename service | `services.common.rename` | POST | `{"json":{"oldProjectName":"x","oldServiceName":"y","newProjectName":"x","newServiceName":"z"}}` |

> **Note**: uses `oldProjectName`/`oldServiceName`, NOT `projectName`/`serviceName`

---

## TRAEFIK (traefik.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| Get dashboard | `traefik.getDashboard` | GET | none |
| Get custom config | `traefik.getCustomConfig` | GET | none |
| Set custom config | `traefik.setCustomConfig` | POST | `{"json":{"config":"..."}}` |
| Get env | `traefik.getEnv` | GET | none |
| Set env | `traefik.setEnv` | POST | `{"json":{...}}` |
| Restart traefik | `traefik.restart` | POST | none |

---

## CERTIFICATES (certificates.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List certificates | `certificates.listCertificates` | GET | none |
| Remove certificate | `certificates.removeCertificate` | POST | `{"json":{"domain":"..."}}` |

---

## NOTIFICATIONS (notifications.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List channels | `notifications.listNotificationChannels` | GET | none |
| Create channel | `notifications.createNotificationChannel` | POST | `{"json":{...}}` |
| Update channel | `notifications.updateNotificationChannel` | POST | `{"json":{...}}` |
| Destroy channel | `notifications.destroyNotificationChannel` | POST | `{"json":{"id":"..."}}` |
| Send test | `notifications.sendTestNotification` | POST | `{"json":{"id":"..."}}` |

---

## DATABASE BACKUPS (databaseBackups.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List backups | `databaseBackups.listDatabaseBackups` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Create backup config | `databaseBackups.createDatabaseBackup` | POST | `{"json":{...}}` |
| Run backup | `databaseBackups.runDatabaseBackup` | POST | `{"json":{...}}` |
| Restore backup | `databaseBackups.restoreDatabaseBackup` | POST | `{"json":{...}}` |
| Update backup | `databaseBackups.updateDatabaseBackup` | POST | `{"json":{...}}` |
| Delete backup | `databaseBackups.deleteDatabaseBackup` | POST | `{"json":{...}}` |
| Get service databases | `databaseBackups.getServiceDatabases` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |

---

## VOLUME BACKUPS (volumeBackups.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List volume backups | `volumeBackups.listVolumeBackups` | GET | `{"json":{...}}` |
| List volume mounts | `volumeBackups.listVolumeMounts` | GET | `{"json":{...}}` |
| Create volume backup | `volumeBackups.createVolumeBackup` | POST | `{"json":{...}}` |
| Run volume backup | `volumeBackups.runVolumeBackup` | POST | `{"json":{...}}` |
| Update volume backup | `volumeBackups.updateVolumeBackup` | POST | `{"json":{...}}` |
| Destroy volume backup | `volumeBackups.destroyVolumeBackup` | POST | `{"json":{...}}` |

---

## DOCKER BUILDERS (dockerBuilders.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List builders | `dockerBuilders.listDockerBuilders` | GET | none |
| Create builder | `dockerBuilders.createDockerBuilder` | POST | `{"json":{...}}` |
| Use builder | `dockerBuilders.useDockerBuilder` | POST | `{"json":{...}}` |
| Stop builder | `dockerBuilders.stopDockerBuilder` | POST | `{"json":{...}}` |
| Remove builder | `dockerBuilders.removeDockerBuilder` | POST | `{"json":{...}}` |

---

## MIDDLEWARES (middlewares.*)

| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| List middlewares | `middlewares.listMiddlewares` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Create middleware | `middlewares.createMiddleware` | POST | `{"json":{...}}` |
| Update middleware | `middlewares.updateMiddleware` | POST | `{"json":{...}}` |
| Destroy middleware | `middlewares.destroyMiddleware` | POST | `{"json":{...}}` |

---

## CLOUDFLARE TUNNEL (cloudflareTunnel.*)

| Action | Procedure | Method |
|--------|-----------|--------|
| Get config | `cloudflareTunnel.getConfig` | GET |
| Set config | `cloudflareTunnel.setConfig` | POST |
| List accounts | `cloudflareTunnel.listAccounts` | GET |
| List tunnels | `cloudflareTunnel.listTunnels` | GET |
| List zones | `cloudflareTunnel.listZones` | GET |
| Get tunnel rules | `cloudflareTunnel.getTunnelRules` | GET |
| Create tunnel rule | `cloudflareTunnel.createTunnelRule` | POST |
| Update tunnel rule | `cloudflareTunnel.updateTunnelRule` | POST |
| Delete tunnel rule | `cloudflareTunnel.deleteTunnelRule` | POST |
| Start tunnel | `cloudflareTunnel.startTunnel` | POST |
| Stop tunnel | `cloudflareTunnel.stopTunnel` | POST |

---

## CLUSTER (cluster.*)

| Action | Procedure | Method |
|--------|-----------|--------|
| List nodes | `cluster.listNodes` | GET |
| Remove node | `cluster.removeNode` | POST |
| Get add worker command | `cluster.addWorkerCommand` | GET |

---

## AUTH (auth.*)

| Action | Procedure | Method |
|--------|-----------|--------|
| Get user | `auth.getUser` | GET |
| Get session | `auth.getSession` | GET |
| Login | `auth.login` | POST |
| Logout | `auth.logout` | POST |

---

## BRANDING (branding.*)

| Action | Procedure | Method |
|--------|-----------|--------|
| Get logo settings | `branding.getLogoSettings` | GET |
| Set logo settings | `branding.setLogoSettings` | POST |
| Get basic settings | `branding.getBasicSettings` | GET |
| Set basic settings | `branding.setBasicSettings` | POST |
| Get interface settings | `branding.getInterfaceSettingsPublic` | GET |
| Get links | `branding.getLinksSettings` | GET |
| Set links | `branding.setLinksSettings` | POST |
| Get custom code | `branding.getCustomCodeSettings` | GET |
| Set custom code | `branding.setCustomCodeSettings` | POST |
| Get error page | `branding.getErrorPageSettings` | GET |
| Set error page | `branding.setErrorPageSettings` | POST |

---

## BACKUP PROVIDERS

### S3/FTP/SFTP/Dropbox/Google/Local
Each provider type has: `createProvider`, `updateProvider`, `deleteProvider`
Some also have: `disconnectProvider`

Namespaces: `dropbox.*`, `google.*`, `ftp.*`, `sftp.*`, `local.*`

---

## BOX SERVICES (services.box.*)

DevBox/workspace containers with IDE support:

| Action | Procedure | Method |
|--------|-----------|--------|
| Init service | `services.box.initService` | POST |
| Inspect | `services.box.inspectService` | GET |
| Start/Stop/Restart/Destroy | `services.box.<action>Service` | POST |
| Git clone | `services.box.gitClone` | POST |
| Run deploy script | `services.box.runDeployScript` | POST |
| Update deploy script | `services.box.updateDeployScript` | POST |
| Update env/resources/etc. | `services.box.update<Setting>` | POST |
| Rebuild Docker image | `services.box.rebuildDockerImage` | POST |
| List/Load presets | `services.box.listPresets` / `loadPreset` | GET/POST |

---

## COMPOSE SERVICES (services.compose.*)

Docker Compose-based services:

| Action | Procedure | Method |
|--------|-----------|--------|
| Inspect | `services.compose.inspectService` | GET |
| Deploy/Start/Stop/Restart/Destroy | `services.compose.<action>Service` | POST |
| Get docker services | `services.compose.getDockerServices` | GET |
| Get issues | `services.compose.getIssues` | GET |
| Update source inline | `services.compose.updateSourceInline` | POST |
| Update source git | `services.compose.updateSourceGit` | POST |
| Update env/basicAuth/maintenance/redirects | `services.compose.update<Setting>` | POST |

---

## WORDPRESS (services.wordpress.*)

Full WordPress management API with ~30+ procedures including plugin/theme management, WP-CLI operations, user management, search-replace, and more.

---

## Setup

After installing this skill, configure your EasyPanel connection in a Claude Code memory file:

```markdown
---
name: EasyPanel Connection
description: EasyPanel server connection details
type: reference
---

- **EasyPanel URL:** https://your-server-ip:3000
- **EasyPanel auth:** Bearer <your-api-token>
```

Then reference that memory file when using the skill.

## Important Notes

- **SSL**: Use `-k` flag for self-signed certs
- **GET queries**: URL-encode the input JSON in `?input=` parameter
- **POST mutations**: Send `{"json": {...}}` as request body
- **Token**: Store in project memory, never hardcode in scripts
- **Zod errors**: When you get a validation error, the `zodErrors` field tells you exactly what fields are missing or wrong — use this to discover schemas
- **Total procedures**: ~250+ tRPC procedures available (extracted from frontend bundle)
- **"services." prefix**: Database and service-type operations use `services.<type>.<action>` pattern. Top-level operations (domains, ports, mounts, etc.) do NOT use the `services.` prefix

---
> Source: [integralmarketingmx/easypanel-api-skill-for-claude](https://github.com/integralmarketingmx/easypanel-api-skill-for-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
