## postulamatic

> - Lenguaje: Python 3.12. Framework: Django.

# === PostulaMatic · Project Rules for Cursor ===
# Objetivo del proyecto (resumen breve)
# - Django app que: (a) permite subir CV, (b) extrae habilidades con IA,
# - inicia sesión en dvcarreras.davinci.edu.ar con credenciales del usuario,
# - scrapea ofertas de https://dvcarreras.davinci.edu.ar/job_board-0.html,
# - calcula "match %" ajustable contra el CV, y
# - si supera umbral, envía emails personalizados (IA) desde la cuenta SMTP del usuario,
# - registra estadísticas y muestra dashboard con tarjetas y detalle.

# === Estándares de código ===
- Lenguaje: Python 3.12. Framework: Django.
- Estilo: seguir Black (line-length 88), isort e importaciones absolutas; compatibilidad con ruff.
- Estructura Django: apps separadas por dominio:
  apps/
    accounts/     (perfil usuario, credenciales externas, email remitente)
    resumes/      (upload/parse de CV)
    jobs/         (scraper cliente, modelos JobPosting, matching)
    outreach/     (generación de email con IA, envío, rate-limit)
    dashboard/    (vistas y API para métricas/tarjetas)
- Pruebas: crear tests por app (pytest o django.test). Añadir tests mínimos con cada feature.

# === Seguridad y secretos ===
- JAMÁS escribir credenciales reales en el repo. Usar variables de entorno (.env) via django-environ.
- No loggear contraseñas, tokens o cookies.
- Para credenciales de usuario (login portal y SMTP) almacenarlas cifradas en DB (Fernet o similar) y desencriptar sólo al usar.
- Respetar Términos del sitio destino. Scraping sólo bajo credenciales y consentimiento explícito del usuario.

# === Scraper (dvcarreras) ===
- Implementar cliente en `apps/jobs/clients/dvcarreras.py` con `requests.Session`.
- Flujo: login (POST) -> mantener cookies -> GET lista (job_board-0.html) -> parsear tarjetas.
- Manejar ofuscación de emails de Cloudflare (`/cdn-cgi/l/email-protection`) y decodificar.
- Respetar límites: backoff con jitter y cabeceras razonables (UA, Referer).
- Diseñar el scraper para reintentos y para fallos parciales sin romper el proceso.

# === Matching ===
- Extraer habilidades del CV (modelo simple primero: TF-IDF/keywords + sinónimos; luego pluggable IA).
- Guardar `match_score` por JobPosting-Resume. Umbral configurable por usuario.
- Exponer una estrategia de matching intercambiable (Strategy pattern) para evolucionar.

# === IA (redacción y personalización) ===
- Aislar prompts en `apps/outreach/prompting.py`.
- Inputs mínimos: descripción del puesto + habilidades detectadas + datos de perfil del usuario.
- Salidas: asunto y cuerpo del email (texto plano y/o HTML simple).
- No incluir datos sensibles que el usuario no haya dado. Evitar alucinaciones (citar datos del anuncio cuando existan).

# === Envío de email ===
- Configurar remitente por usuario: host, puerto, TLS/SSL, usuario/clave (cifrados en DB).
- Colas y rate limit: planificar Celery (Redis) o, si no está disponible, tareas periódicas con locks y sleep con jitter.
- Límite diario por usuario configurable; pausas aleatorias entre envíos.

# === Dashboard ===
- Tarjetas por postulación: estado (enviado, reintento, error), fecha, empresa, posición, match%.
- Vista detalle con: email enviado (preview), CV/adaptaciones, logs de envío y scraper.
- API JSON para front (siendo server-rendered o HTMX inicialmente).

# === Migraciones y datos ===
- Crear modelos con migraciones atómicas. No romper datos existentes.
- Añadir fixtures de desarrollo cuando corresponda (datos sintéticos).

# === Git y CI ===
- No tocar Dockerfile/compose/CI sin solicitud explícita.
- Commits pequeños, con mensajes claros. Donde cambie esquema, explicar migraciones.

# === Archivos y rutas esperadas ===
- Este repo ya contiene: Dockerfile, docker-compose.yml, .pre-commit-config.yaml.
- Mantener `/static/` y `collectstatic` funcionando.
- Evitar cambios en Nginx/infra desde Cursor salvo que se pida.

# === Pedidos a Cursor (meta) ===
- Antes de cambios multi-archivo, proponer plan y diff.
- Referenciar rutas concretas y crear archivos donde falten.
- Al crear nuevas apps: añadir a INSTALLED_APPS y urls.py si aplica.
- Siempre incluir pruebas mínimas y/o comandos de verificación manual.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/idgleb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
