## erp-ace

> Cuando el usuario diga cualquiera de estas palabras:


Cuando el usuario diga cualquiera de estas palabras:
- "deploy", "desplegar", "subir", "publicar", "actualizar producción"

---

## DEPLOY UNIVERSAL - Proceso Completo

Este proceso despliega TODO: código y base de datos (si hay cambios).

### Configuración

```bash
# Producción
PROD_HOST="144.126.150.120"
PROD_DB_PORT="15433"
PROD_DB_USER="erp_user"
PROD_DB_PASS="erp_password_2024"
PROD_DB_NAME="erp_db"
SSH_HOST="144"
REMOTE_PATH="/var/proyectos/erp_ace"

# Local
LOCAL_PATH="/home/diazhh/dev/erp"
BACKUP_DIR="/home/diazhh/dev/erp/backups"
```

---

### Paso 1: Verificaciones Previas

```bash
# Verificar SSH
ssh -q 144 exit && echo "✓ SSH OK" || echo "✗ SSH FAILED"

# Verificar rama
cd /home/diazhh/dev/erp && git branch --show-current
```

**IMPORTANTE**: Debe estar en rama `main`. Si no, abortar.

---

### Paso 2: Commit y Push de Cambios Locales

```bash
cd /home/diazhh/dev/erp && git status --porcelain
```

**Si hay cambios**, ejecutar:
```bash
git add .
git commit -m "Deploy $(date '+%Y-%m-%d %H:%M:%S')"
git push origin main
```

---

### Paso 3: Pull en el Servidor

```bash
ssh 144 "cd /var/proyectos/erp_ace && git fetch origin main && git pull origin main"
```

Verificar sincronización:
```bash
cd /home/diazhh/dev/erp && git rev-parse --short HEAD
ssh 144 "cd /var/proyectos/erp_ace && git rev-parse --short HEAD"
```

**Si no coinciden**, forzar:
```bash
ssh 144 "cd /var/proyectos/erp_ace && git fetch origin && git reset --hard origin/main"
```

---

### Paso 4: Verificar Migraciones Pendientes

**DESPUÉS del pull**, verificar si hay migraciones nuevas:

```bash
ssh 144 "cd /var/proyectos/erp_ace/backend && npx sequelize-cli db:migrate:status 2>&1 | grep -E '^(up|down)'"
```

Contar cuántas dicen "down":
- Si hay migraciones "down" → HAY CAMBIOS DE BD, continuar con Paso 5
- Si todas dicen "up" → NO hay cambios de BD, saltar al Paso 7

---

### Paso 5: Backup de BD de Producción (SOLO si hay migraciones)

```bash
mkdir -p /home/diazhh/dev/erp/backups
PGPASSWORD=erp_password_2024 pg_dump -h 144.126.150.120 -p 15433 -U erp_user -d erp_db --no-owner --no-acl -F c -f /home/diazhh/dev/erp/backups/pre_deploy_$(date +%Y%m%d_%H%M%S).sql
```

Verificar:
```bash
ls -lh /home/diazhh/dev/erp/backups/*.sql | tail -1
```

---

### Paso 6: Ejecutar Migraciones en Producción (SOLO si hay pendientes)

```bash
ssh 144 "cd /var/proyectos/erp_ace/backend && NODE_ENV=production npx sequelize-cli db:migrate"
```

Verificar que todas quedaron "up":
```bash
ssh 144 "cd /var/proyectos/erp_ace/backend && npx sequelize-cli db:migrate:status 2>&1 | grep -E '^(up|down)'"
```

---

### Paso 7: Dependencias del Backend

```bash
ssh 144 "cd /var/proyectos/erp_ace/backend && yarn install --production 2>/dev/null || npm install --omit=dev"
```

---

### Paso 8: Build del Frontend

```bash
ssh 144 "cd /var/proyectos/erp_ace/frontend && yarn install && yarn build 2>/dev/null || (npm install && npm run build)"
```

---

### Paso 9: Reiniciar Servicios PM2

```bash
ssh 144 "pm2 restart erp-backend erp-frontend"
```

Esperar 3 segundos y verificar:
```bash
ssh 144 "pm2 list | grep erp"
```

Health check:
```bash
ssh 144 "curl -s http://localhost:5003/health"
```

---

### Paso 10: Sincronizar BD de Producción a Local

**SIEMPRE** al final del deploy, traer la BD actualizada a local:

```bash
# Descargar BD de producción
PGPASSWORD=erp_password_2024 pg_dump -h 144.126.150.120 -p 15433 -U erp_user -d erp_db --no-owner --no-acl -F c -f /home/diazhh/dev/erp/backups/post_deploy_$(date +%Y%m%d_%H%M%S).sql

# Restaurar en local (terminar conexiones primero)
BACKUP_FILE=$(ls -t /home/diazhh/dev/erp/backups/post_deploy_*.sql | head -1)
docker cp $BACKUP_FILE erp_postgres:/tmp/restore.sql
docker exec erp_postgres psql -U erp_user -d postgres -c "UPDATE pg_database SET datallowconn = false WHERE datname = 'erp_db';"
docker exec erp_postgres psql -U erp_user -d postgres -c "SELECT pg_terminate_backend(pid) FROM pg_stat_activity WHERE datname = 'erp_db';"
docker exec erp_postgres psql -U erp_user -d postgres -c "DROP DATABASE IF EXISTS erp_db;"
docker exec erp_postgres psql -U erp_user -d postgres -c "CREATE DATABASE erp_db OWNER erp_user;"
docker exec erp_postgres pg_restore -U erp_user -d erp_db --no-owner --no-acl /tmp/restore.sql
docker exec erp_postgres rm /tmp/restore.sql
```

Verificar:
```bash
docker exec erp_postgres psql -U erp_user -d erp_db -c "SELECT count(*) FROM information_schema.tables WHERE table_schema = 'public';"
```

---

## Información del Servidor

| Parámetro | Valor |
|-----------|-------|
| **SSH Alias** | `144` |
| **IP** | 144.126.150.120 |
| **BD Producción** | 144.126.150.120:15433 |
| **BD Local** | localhost:5433 (Docker: erp_postgres) |
| **BD Usuario** | erp_user |
| **BD Password Prod** | erp_password_2024 |
| **BD Password Local** | erp_password_dev_2024 |
| **PM2 Backend** | erp-backend (puerto 5003) |
| **PM2 Frontend** | erp-frontend (puerto 5004) |
| **Dominio** | https://erp.atilax.io |

---

## Al Finalizar Deploy

Informar al usuario:

1. ✅ o ❌ Estado de cada paso
2. 📦 Backup creado (si hubo migraciones)
3. 🔄 Migraciones ejecutadas (cantidad)
4. 📝 Hash del commit desplegado
5. 🟢 Estado de servicios PM2
6. 🔄 BD local sincronizada
7. 🔗 URLs:
   - **Backend**: http://144.126.150.120:5003
   - **Frontend**: http://144.126.150.120:5004
   - **Dominio**: https://erp.atilax.io

---

## SINCRONIZAR BD (sin deploy)

Cuando el usuario diga: "sincronizar", "sync", "bajar bd", "actualizar local"

```bash
# Descargar
mkdir -p /home/diazhh/dev/erp/backups
PGPASSWORD=erp_password_2024 pg_dump -h 144.126.150.120 -p 15433 -U erp_user -d erp_db --no-owner --no-acl -F c -f /home/diazhh/dev/erp/backups/prod_sync_$(date +%Y%m%d_%H%M%S).sql

# Restaurar
BACKUP_FILE=$(ls -t /home/diazhh/dev/erp/backups/prod_sync_*.sql | head -1)
docker cp $BACKUP_FILE erp_postgres:/tmp/restore.sql
docker exec erp_postgres psql -U erp_user -d postgres -c "DROP DATABASE IF EXISTS erp_db;"
docker exec erp_postgres psql -U erp_user -d postgres -c "CREATE DATABASE erp_db OWNER erp_user;"
docker exec erp_postgres pg_restore -U erp_user -d erp_db --no-owner --no-acl /tmp/restore.sql
docker exec erp_postgres rm /tmp/restore.sql

# Verificar
docker exec erp_postgres psql -U erp_user -d erp_db -c "SELECT count(*) FROM information_schema.tables WHERE table_schema = 'public';"
```

---

## Rollback de Emergencia

Si algo sale mal:

### Listar backups
```bash
ls -lh /home/diazhh/dev/erp/backups/*.sql
```

### Restaurar backup en producción
```bash
# Seleccionar archivo de backup (reemplazar ARCHIVO)
BACKUP="/home/diazhh/dev/erp/backups/ARCHIVO.sql"

# Detener backend
ssh 144 "pm2 stop erp-backend"

# Restaurar
PGPASSWORD=erp_password_2024 pg_restore -h 144.126.150.120 -p 15433 -U erp_user -d erp_db --clean --no-owner --no-acl $BACKUP

# Reiniciar
ssh 144 "pm2 start erp-backend"
```

---

## Comandos Útiles

```bash
# Ver logs
ssh 144 "pm2 logs erp-backend --lines 50"

# Estado PM2
ssh 144 "pm2 list"

# Verificar commits
git log -1 --oneline
ssh 144 "cd /var/proyectos/erp_ace && git log -1 --oneline"

# Forzar sync git
ssh 144 "cd /var/proyectos/erp_ace && git fetch origin && git reset --hard origin/main"

# Estado migraciones local
cd /home/diazhh/dev/erp/backend && npx sequelize-cli db:migrate:status

# Estado migraciones producción
ssh 144 "cd /var/proyectos/erp_ace/backend && npx sequelize-cli db:migrate:status"
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/diazhh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
