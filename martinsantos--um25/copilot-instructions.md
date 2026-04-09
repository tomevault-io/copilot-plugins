## um25

> **BASELINE**: `v0.0.1-production-baseline`

# 🔒 REGLAS INMUTABLES - ARQUITECTURA SERVIDOR PRODUCCIÓN

**BASELINE**: `v0.0.1-production-baseline`
**SERVIDOR**: `23.105.176.45`
**FECHA**: 2025-11-28

---

## ⚠️ REGLA DE ORO

**PRODUCCIÓN ES READ-ONLY. TODO CAMBIO VÍA GIT → CI/CD**

---

## 🚫 PROHIBICIONES ABSOLUTAS

### **NUNCA Ejecutar en Producción**

```bash
❌ git pull (manual)
❌ git merge
❌ git reset --hard
❌ npm install (sin CI/CD)
❌ editar .env directamente
❌ pm2 delete
❌ docker-compose down (PostgreSQL)
❌ chmod 777
❌ rm -rf logs/
❌ push directo a master
```

---

## ✅ STACK INMUTABLE

```yaml
Frontend: Astro 5.7.4 (SSR) + PM2
Backend: Directus 10.8.3
Database: PostgreSQL 15 (Docker:5432)
Proxy: Nginx (80/443)
Node: 18.x
```

**NO actualizar versiones sin: testing + backup + plan rollback**

---

## 📐 ARQUITECTURA FIJA

```
Internet
    ↓
Nginx (80/443)
    ↓
PM2: astro-ultimamilla (4321)
    ↓
Directus API (interno)
    ↓
PostgreSQL (Docker:5432)
```

**NO modificar esta estructura**

---

## 🔄 FLUJO OBLIGATORIO

```
Local → feature/branch → PR → develop → merge
    ↓
develop → PR → master → GitHub Actions → DEPLOY
```

**BYPASS = VIOLACIÓN**

---

## 🛡️ ARCHIVOS PROTEGIDOS

```bash
# NUNCA editar directamente:
.env
directus-admin/.env
astro.config.mjs
ecosystem.config.js
/etc/nginx/nginx.conf

# Modificar SOLO vía Git → CI/CD
```

---

## 🚨 PROTOCOLO DE EMERGENCIA

### **Sitio Caído**

```bash
1. ssh ultimamilla
2. pm2 restart astro-ultimamilla
3. Si falla:
   git checkout v0.0.1-production-baseline
   pm2 restart astro-ultimamilla
4. Verificar: curl -I https://www.ultimamilla.com.ar
5. Reportar en GitHub Issues
```

**NO intentar "fix en caliente"**

---

## 📊 MONITOREO ACTIVO

```bash
# Cron jobs (NO DESACTIVAR):
*/5 * * * * /root/scripts/health-check.sh
0 * * * * /root/scripts/server-metrics.sh
```

**Logs obligatorios:**
- `/var/log/health-check.log`
- `/root/.pm2/logs/astro-ultimamilla-*.log`

---

## ✅ COMANDOS PERMITIDOS

```bash
# Solo lectura (OK):
pm2 list
pm2 logs
docker ps
git status
git log
df -h
free -h

# Con precaución:
pm2 restart astro-ultimamilla (solo emergencia)

# PROHIBIDO sin aprobación:
Todo lo demás
```

---

## 💾 BACKUP OBLIGATORIO

**Antes de cualquier cambio mayor:**

```bash
BACKUP_DIR=~/backups/manual-$(date +%Y%m%d_%H%M%S)
mkdir -p $BACKUP_DIR
cd /root/fumbling-field
tar -czf $BACKUP_DIR/backup.tar.gz \
  --exclude='node_modules' --exclude='.git' --exclude='dist' .
```

**Sin backup = NO proceder**

---

## 🎯 SERVICIOS CRÍTICOS (24/7)

```
✅ www.ultimamilla.com.ar
✅ sgi.ultimamilla.com.ar
✅ www.umbot.com.ar
✅ viveroloscocos.com.ar
✅ CyberPanel (23.105.176.45:8090)
```

**Downtime NO planificado = VIOLACIÓN CRÍTICA**

---

## 🔑 VARIABLES SECRETAS

```bash
# NUNCA commitear a Git:
DATABASE_URL
DIRECTUS_KEY
DIRECTUS_SECRET
PUBLIC_DIRECTUS_TOKEN
DB_PASSWORD
SECRET
ADMIN_PASSWORD
```

**Secrets SOLO en servidor (.env local)**

---

## 📋 CHECKLIST PRE-CAMBIO

```
[ ] Backup reciente (< 24h)?
[ ] Testeado en develop?
[ ] Plan de rollback?
[ ] Monitoreo activo?
[ ] Documentado en GitHub?
[ ] Equipo notificado?

NO a cualquiera = NO proceder
```

---

## ⚖️ NIVELES DE ESCALACIÓN

```yaml
🟡 Advertencia (CPU/RAM > 80%):
   → Monitorear logs

🟠 Crítico (Servicio degradado):
   → GitHub Issue + investigar

🔴 Emergencia (Servicio caído):
   → Rollback a baseline INMEDIATO
   → Incident report

⚫ Desastre (Múltiples servicios caídos):
   → NO tocar nada
   → Contactar Tech Lead URGENTE
```

---

## 🎓 DOCUMENTACIÓN OBLIGATORIA

**Leer ANTES de acceder a producción:**

1. `REGLAS_ARQUITECTURA_SERVIDOR.md` (completo)
2. `WORKFLOW_GITFLOW.md` (flujo trabajo)
3. `IMPLEMENTATION_COMPLETE.md` (guía completa)

---

## 🔐 LAS 5 REGLAS DE ORO

```
1. BASELINE ES SAGRADO (v0.0.1-production-baseline)
2. GIT FLOW ES LEY (feature → develop → master)
3. PRODUCCIÓN ES READ-ONLY (solo CI/CD escribe)
4. BACKUP BEFORE CHANGE (sin backup = no change)
5. ROLLBACK OVER FIX (emergencia = rollback primero)
```

---

## ✍️ COMMIT DE CUMPLIMIENTO

```
Al acceder al servidor de producción, acepto:

✓ Seguir estas reglas sin excepción
✓ NO modificar archivos directamente en producción
✓ TODO cambio vía Git Flow → CI/CD
✓ Ejecutar protocolo de rollback en emergencias
✓ Mantener backups antes de cambios mayores

Violación = Remoción de acceso SSH
```

---

**VERSIÓN**: 1.0
**ESTADO**: ✅ ACTIVO Y OBLIGATORIO
**MODIFICABLE**: ❌ NO (requiere aprobación Tech Lead)

---

## 📞 CONTACTO EMERGENCIA

```bash
# Ver estado servicios:
ssh ultimamilla "pm2 list && docker ps"

# Ver logs recientes:
ssh ultimamilla "tail -50 /var/log/health-check.log"

# Rollback de emergencia:
ssh ultimamilla "cd /root/fumbling-field && \
  git checkout v0.0.1-production-baseline && \
  pm2 restart astro-ultimamilla"
```

---

**ESTE DOCUMENTO DEBE ESTAR VISIBLE EN TODO MOMENTO**
**IMPRIMIR Y MANTENER CERCA DEL WORKSPACE**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martinsantos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
