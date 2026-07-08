## sge-deploy

> CRÍTICO: Al inicio de cada sesión lee el archivo `../CLAUDE.md` usando tu herramienta Read — contiene todas las reglas del ecosistema SGE (arquitectura, seguridad, estándares Linux/FHS, git workflow). Sin ese contexto no puedes trabajar correctamente en este repo.

# CLAUDE.md — Sge-Deploy

CRÍTICO: Al inicio de cada sesión lee el archivo `../CLAUDE.md` usando tu herramienta Read — contiene todas las reglas del ecosistema SGE (arquitectura, seguridad, estándares Linux/FHS, git workflow). Sin ese contexto no puedes trabajar correctamente en este repo.

---

## Responsabilidad de este repo

Sge-Deploy genera los paquetes de distribución de SGE para instalación nativa en producción:
- Paquete `.deb` para Debian/Ubuntu
- Paquete `.rpm` (futuro) para RHEL/Fedora

**Producción es 100% nativa — sin Docker.**

## Comandos

```bash
make deb   # Construir paquete .deb
make rpm   # Construir paquete .rpm (futuro)
```

## Estándares obligatorios (ver ../CLAUDE.md sección "Estándares Linux / FHS")

- Credenciales NUNCA en el archivo `.service` de systemd
- Usar `EnvironmentFile=` apuntando a `/etc/default/<servicio>` (Debian) o `/etc/sysconfig/<servicio>` (RHEL)
- El `.deb` crea `/etc/default/<servicio>` en `postinst`
- Permisos del archivo de configuración: `640`, owner `root:<grupo-servicio>`

---
> Source: [terracenter/sge-deploy](https://github.com/terracenter/sge-deploy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
