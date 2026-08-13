## clinical

> Notas para agentes que trabajan en este repo. Contenido espejo de `CLAUDE.md`.

# AGENTS.md

Notas para agentes que trabajan en este repo. Contenido espejo de `CLAUDE.md`.

## Deploy — qué rama usar <!-- /aprende 2026-07-28 -->

- El deploy **es** un push a `main`: CodePipeline observa esa rama. La regla global "nunca commitear a main" aplica al trabajo en curso, no al deploy; para publicar hay que llegar a `main`. Si el cambio es grande o arriesgado, trabajarlo en `feat/`/`fix/` y mergear a `main` para desplegar. <!-- /aprende 2026-07-28 -->
- Cambio **sólo de frontend** → `git push origin main` y listo; lo recoge `clinical-frontend-pipeline` (Source → Build, ~2 min). **No** hace falta `sam deploy`. <!-- /aprende 2026-07-28 -->
- Cambio de **backend** o `template.yaml` → además `sam deploy` + `scripts/fix-apigw-after-deploy.sh`. <!-- /aprende 2026-07-28 -->
- Verificar el deploy por **contenido**, no por hash: CodeBuild compila con otras env vars, así que el nombre del bundle difiere del build local aunque el commit sea el mismo. Buscar en el JS servido un string que sólo exista en el fix. <!-- /aprende 2026-07-28 -->

---
> Source: [mserranolm/clinical](https://github.com/mserranolm/clinical) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
