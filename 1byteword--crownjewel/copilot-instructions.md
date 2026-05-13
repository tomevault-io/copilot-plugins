## crownjewel

> Operational reference for this repo. Audience: future-you and any AI agents

# AGENTS.md

Operational reference for this repo. Audience: future-you and any AI agents
helping out (Cursor, Claude, Codex). Read this before making changes; keep it
honest if you change the architecture.

## What this site is

- **Two-page** static site behind nginx in Kubernetes.
- `index.html` — homepage. Bricolage Grotesque + an animated dithered palm‑tree canvas.
  Has a single Google Font (`Bricolage Grotesque`, preconnected + preloaded) and an
  inline `<script>` that drives the canvas. Renders the scene at dither
  resolution, reuses one offscreen buffer, throttles to 24 fps, pauses when
  offscreen / tab hidden / `prefers-reduced-motion`.
- `writing/index.html` — blog index, served at `/writing`. Bricolage Grotesque,
  same aesthetic as the homepage. The `publish` script inserts entries between
  the `<!-- POST_ENTRIES_START -->` / `<!-- POST_ENTRIES_END -->` markers and
  keeps them sorted newest-first.
- `writing/<slug>.html` — standalone post pages, one per published post, served
  at `/writing/<slug>` (extensionless). Filenames are derived from the source
  markdown with the `YYYY-MM-DD-` prefix stripped.

> Historical note: an earlier iteration of this repo aimed for a single
> ~3 KB `index.html` with zero JS and no fonts. That constraint is **retired**
> as of the Bricolage Grotesque / canvas redesign. Don't reintroduce it without
> explicit intent.

## Deploy pipeline (current)

```
git push main
   │
   ├─► .github/workflows/build.yaml
   │       buildx multi-arch (amd64 + arm64)
   │       push :main-<sha>, :latest, :canary
   │       Trivy scan, upload SARIF
   │
   └─► ArgoCD watches k8s/  ──►  syncs Deployment/Service/Ingress
                                  (only when k8s/ manifests change)
```

### Known gotcha: image refresh

`k8s/deployment.yaml` references `:latest` with `imagePullPolicy: Always`.
ArgoCD only syncs when something in `k8s/` changes — pushing a new HTML edit
updates the registry tag but **does not** restart the running pods. To pick
up a new image after a content-only push, run:

```bash
kubectl rollout restart deployment personal-site
kubectl rollout status  deployment personal-site
```

The clean fix is to pin the deployment to a SHA tag and have CI bump that
tag in `k8s/kustomization.yaml` after the build. Until then, remember the
rollout-restart step.

## Make targets

```bash
make dev            # python3 -m http.server 8000

make build          # docker build -t ghcr.io/1byteword/crownjewel:<sha>,:latest
make push           # build + push :latest and :<sha>

make deploy         # kubectl apply -f k8s/argocd-application.yaml
make status         # kubectl get pods,svc,ingress -l app=personal-site

make canary-build   # buildx --platform linux/amd64 -t :canary
make canary-test    # run :canary locally on :8080
make canary-push    # push :canary tag
make canary-status  # kubectl get pods,svc,ingress -l app=personal-site-canary
make promote        # retag :canary → :latest + :<sha>, push both

make clean          # rmi local images
```

## Canary workflow

Two independent stacks share the same cluster:

| Env       | URL                       | Tag       | Replicas |
|-----------|---------------------------|-----------|----------|
| prod      | `azhankhan.com`           | `:latest` | 2        |
| canary    | `preview.azhankhan.com`   | `:canary` | 1        |

Typical loop:

```bash
# edit index.html / writing/index.html / writing/<slug>.html / etc.
make canary-push                       # ships preview.azhankhan.com
# review at https://preview.azhankhan.com
make promote                           # canary → latest
kubectl rollout restart deployment personal-site   # see "Known gotcha"
git add . && git commit -m "..." && git push
```

## Publishing posts

```bash
# 1. Write src/posts/YYYY-MM-DD-slug.md
#    Front matter is implicit: first H1 is the title, first paragraph is the
#    excerpt (auto-truncated at 200 chars). Date comes from the filename, or
#    pass --date YYYY-MM-DD.

# 2. Publish
./publish src/posts/YYYY-MM-DD-slug.md

# Side effects:
#   - writing/<slug>.html created (date prefix stripped from filename; URL is
#     extensionless: /writing/<slug>)
#   - <article> entry inserted into writing/index.html, sorted by <time> desc,
#     dedup-by-slug on re-publish
#   - Homepage 03/writing block re-mirrored with the N newest posts
#     (HOMEPAGE_LATEST_COUNT in publish, default 5)
#   - Any ![](image.jpg) refs auto-copied into img/ with paths rewritten
```

Supported markdown subset: `#`/`##`/`###`, `**bold**`, `*italic*`, `` `code` ``,
fenced code blocks, `[text](url)`, `![alt](image.jpg)`, `- bullet` lists.
A paragraph that is just `*italic*` becomes a `<p class="attribution">`.

## Kubernetes layout (`k8s/`)

| File                          | Purpose                                       |
|-------------------------------|-----------------------------------------------|
| `argocd-application.yaml`     | ArgoCD `Application`, watches `k8s/` on main  |
| `deployment.yaml`             | Prod Deployment (2 replicas, `:latest`)       |
| `service.yaml`                | Prod ClusterIP                                |
| `ingress.yaml`                | Prod ingress for `azhankhan.com`              |
| `canary-deployment.yaml`      | Canary Deployment (1 replica, `:canary`)      |
| `canary-service.yaml`         | Canary ClusterIP                              |
| `canary-ingress.yaml`         | Canary ingress for `preview.azhankhan.com`    |
| `cert-issuer.yaml`            | cert-manager `ClusterIssuer` (Let's Encrypt)  |
| `kustomization.yaml`          | Bundles the above for ArgoCD                  |

Resource requests are deliberately tiny (10m CPU / 16 Mi memory per replica;
limits 100m / 64 Mi). The image runs as the nginx user (UID 101) with all
caps dropped and the RuntimeDefault seccomp profile.

## nginx config (`nginx.conf`)

- `listen 8080` (so it can run as non-root)
- `try_files $uri $uri/ /index.html`
- `Cache-Control: public,must-revalidate` + `expires 1h` for HTML
- `gzip on; gzip_comp_level 6;`
- `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`
- `server_tokens off`, `access_log off`

## Cost

DigitalOcean smallest cluster (~$12/month) + domain (~$12/year) +
free Let's Encrypt certs. Negligible egress for static HTML.

## First-time deploy (reference)

1. Create GHCR token, `docker login ghcr.io -u 1byteword`.
2. Make the `crownjewel` package public on GitHub.
3. Point `azhankhan.com` and `preview.azhankhan.com` A records at the
   ingress controller external IP.
4. Ensure `cert-manager`, `nginx-ingress`, and ArgoCD are installed.
5. `make deploy` to install the ArgoCD `Application`.
6. Wait ~2–5 min for cert-manager to issue both TLS certs.

## House style

- Don't add a build step. Static HTML with inline assets only.
- Don't reintroduce the "single ~3 KB file / no JS / no fonts" doctrine
  unless explicitly resurrecting it.
- Every external dependency is a deliberate choice — currently: one Google
  Font, one nginx base image. Don't add more without a reason.
- When editing this file, keep it honest: if you change the architecture,
  update the relevant section here.

---
> Source: [1byteword/crownjewel](https://github.com/1byteword/crownjewel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
