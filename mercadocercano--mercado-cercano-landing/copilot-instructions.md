## mercado-cercano-landing

> Landing de conversión para Mercado Cercano, priorizando Server Components.

# CLAUDE.md — mercado-cercano-landing

Landing de conversión para Mercado Cercano, priorizando Server Components.

## Stack y puerto

- Next.js 14, App Router
- Desarrollo: puerto **3007**

## Comandos

```bash
npm install
npm run dev
```

## Estructura

- `app/` enrutamiento; `components/` UI; `lib/` (`site.ts`, `env.ts`)

## Producción y despliegue

- Sitio: https://mercadocercano.com.ar
- Kubernetes: manifiestos en `k8s/landing/` (deployment, service, ingress con Traefik y cert-manager)
- Imagen (ejemplo): `docker buildx build --platform linux/amd64 -t registry.digitalocean.com/mc-bootstrap-registry/mercado-cercano-landing:v0.1.0 --push .`
- Aplicar: `kubectl apply -f k8s/landing/` y `kubectl rollout restart deployment/mercado-cercano-landing`

## Documentación interna

- `LANDING_PRODUCCION_VERIFICADA.md`
- `PIPELINE_DEPLOY_LANDING.md`

Hablar siempre en español en este repositorio.

---
> Source: [mercadocercano/mercado-cercano-landing](https://github.com/mercadocercano/mercado-cercano-landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
