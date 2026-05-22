## winenot

> Ce fichier definit les regles de travail pour les agents IA sur ce repository.

# CLAUDE.md

Ce fichier definit les regles de travail pour les agents IA sur ce repository.

## Objectif du projet
WineNot est une application de gestion de cave a vin avec:
- Backend Python/FastAPI
- Base MySQL
- Frontend PHP
- Deploiement Docker Compose et Kubernetes (Helm + ArgoCD)

## Structure utile
- `backend/code/`: API FastAPI, models SQLAlchemy, routes
- `backend/tests/`: tests backend (pytest)
- `frontend/code/`: pages PHP
- `db/sql/`: scripts init DB
- `kube/winenot/`: chart Helm K8s
- `deployment/`: playbooks Ansible pour ArgoCD
- `docker-compose.yaml`: stack locale

## Regles generales de code
- Faire des changements minimaux et cibles.
- Ne pas renommer API/routes/schemas sans raison explicite.
- Preserver le style existant du fichier (indentation, nommage, organisation).
- Eviter les refactors larges non demandes.
- Ajouter des commentaires seulement si un bloc est difficile a comprendre.
- Garder la compatibilite avec Python 3.11.

## Regles Backend (FastAPI)
- Entree API: `backend/code/main.py`.
- Routes dans `backend/code/routes/`.
- Utiliser SQLAlchemy/SQLModel deja presents, ne pas introduire un autre ORM.
- Toute nouvelle route doit:
  - valider les entrees via schemas
  - gerer les erreurs HTTP proprement
  - respecter le pattern auth/permissions existant
- Eviter les secrets hardcodes dans le code.

## Regles Frontend (PHP)
- Conserver la structure existante par domaine (`auth/`, `bottles/`, `cellars/`, etc.).
- Eviter de dupliquer la logique backend dans le frontend.
- Toute URL API doit passer par la config (`frontend/code/config/`).

## Regles DB
- Les changements schema doivent etre coherents entre:
  - models Python
  - scripts SQL dans `db/sql/` si necessaire
- Ne jamais supprimer des donnees de prod dans un script sans demande explicite.

## Tests et validation
Avant de proposer un merge, lancer selon le scope:
- Lint/tests backend si concernes
- Tests unitaires/integ existants
- Verifier que les endpoints modifies repondent

Commandes frequentes:
- `docker compose up --build`
- `cd backend && pytest -v`
- `helm lint kube/winenot`
- `helm template winenot kube/winenot`

## Regles Kubernetes / ArgoCD / K3s
- Chart principal: `kube/winenot/`.
- Garder les valeurs Helm coherentes (images, ports, env vars, ingress, TLS).
- Eviter les tags d image `latest` en production.
- Pour Ingress/cert-manager:
  - verifier `ingress.hosts`
  - verifier secret TLS
  - verifier issuer letsencrypt
- Toute modif ArgoCD doit conserver un manifest applicable idempotent.

## Securite
- Ne jamais commiter de mots de passe reels, tokens, cles privees.
- Preferer variables d environnement et secrets K8s.
- Eviter les logs contenant credentials ou JWT complets.

## Workflow de contribution IA
1. Lire les fichiers impactes avant modification.
2. Proposer la plus petite correction qui resout le probleme.
3. Mettre a jour docs/manifests si comportement change.
4. Verifier rapidement l impact (tests/lint/template helm).
5. Donner un resume clair des changements et des limites.

## A eviter
- Modifier des fichiers non lies a la demande.
- Ajouter de nouvelles dependances lourdes sans justification.
- Casser la compatibilite Docker/K8s existante.
- Faire des operations destructives (drop/reset) sans demande explicite.

## Si information manquante
Poser des questions courtes et precises avant de faire une hypothese risquee:
- environnement cible (local, VPS, k3s)
- namespace/host/domain
- image registry et tags
- niveau attendu (dev vs production)

---

## Diagnostic TLS/Let's Encrypt (MAI 2026)

### Contexte VPS K3s
- **VPS**: lvp.ovh (54.37.229.190), Debian 13, K3s v1.34.3+k3s1
- **Ingress Controller**: Traefik v3.5.1 (port 80:32552, 443:30870)
- **cert-manager**: v1.15.1 (en cluster-manager namespace)
- **Problème initial**: ACME HTTP-01 challenges retouraient 404 => Let's Encrypt rejetait validation
- **Root cause**: Multiples problèmes cumulates:
  1. **ClusterIssuer letsencrypt-prod n'existait pas** (ingress le référençait mais il n'était pas créé)
  2. **Solver ingress créé par cert-manager** n'avait pas `ingressClassName: traefik`
  3. **Apache système** écoutant port 80 bloquait Traefik HTTP solver
  4. **Routing Traefik** envoyait requêtes ACME au backend au lieu du solver

### Solution implémentée
1. **Créé ClusterIssuer letsencrypt-prod** avec HTTP-01 solver configuré pour Traefik
2. **Créé ClusterIssuer letsencrypt-staging** pour tests sans rate limits
3. **Créé secret TLS auto-signé** `lvp-ovh-tls-selfsigned` en attendant Let's Encrypt
4. **Mis à jour Ingress** pour utiliser secret self-signed pour HTTPS immédiat
5. **À faire**: 
   - Arrêter Apache système (sudo systemctl stop apache2 / disable)
   - Forcer cert-manager à recréer challenges avec issuer corrigé
   - Valider Let's Encrypt validation HTTP-01 

### Tests Let's Encrypt validés
- ✅ ClusterIssuer letsencrypt-prod créé
- ✅ ClusterIssuer letsencrypt-staging créé
- ✅ Ingress routing HTTP/HTTPS vers backend
- ✅ HTTPS fonctionne avec self-signed cert
- ⚠️ ACME HTTP-01 validation bloquée (Apache vs Traefik sur port 80)

### Commandes clés pour diagnostic/réparation
```bash
# Vérifier issuers
kubectl get clusterissuer -o wide

# Vérifier challenges ACME
kubectl -n winenot get challenge,order -o wide

# Vérifier cert-manager logs
kubectl logs -n cert-manager -l app.kubernetes.io/name=cert-manager --tail=50

# Tester HTTP-01 endpoint
curl -v http://lvp.ovh/.well-known/acme-challenge/test

# Vérifier Traefik services
kubectl -n kube-system get svc traefik -o wide

# Arrêter Apache (blocker)
sudo systemctl stop apache2
sudo systemctl disable apache2

# Forcer renouvellement certs
kubectl -n winenot delete certificate --all
# cert-manager les recréera avec les issuers correctement en place
```

### État actuel (9 mai 2026, 15:07 UTC)
- Backend: ✅ Déployé, pods Running
- MySQL: ✅ Déployé, pods Running
- HTTPS (self-signed): ✅ Fonctionnel
- Let's Encrypt (HTTP-01): ⏳ En attente de correction Apache port 80

### SOLUTION DÉPLOIEMENT

**État stable avec certificat auto-signé:**
1. Ingress pointe vers secret `lvp-ovh-tls-selfsigned` (créé manuellement)
2. HTTPS fonctionne avec cert auto-signé (navigateurs afficheront erreur de cert invalide)
3. ClusterIssuer letsencrypt-prod et letsencrypt-staging créés et prêts

**Pour activer Let's Encrypt en production:**

Prérequis:
```bash
# 1. ARRÊTER APACHE qui bloque port 80 (CRITIQUE)
ssh -p 7022 debian@lvp.ovh
sudo systemctl stop apache2
sudo systemctl disable apache2

# 2. Vérifier que Traefik expose port 80
kubectl -n kube-system get svc traefik
# Doit montrer port 80 ouvert au public (LoadBalancer 54.37.229.190)

# 3. Modifier values.yaml
# Changer: ingress.tls.secretName: lvp-ovh-tls-selfsigned -> lvp-ovh-tls
# (cert-manager créera ce secret automatiquement après validation ACME)

# 4. Force redéploiement
kubectl -n winenot delete certificate --all
kubectl -n winenot delete certificaterequest --all
kubectl -n winenot delete secret lvp-ovh-tls 2>/dev/null || true

# 5. Re-appliquer Helm chart ou ArgoCD sync
helm upgrade --install winenot kube/winenot -n winenot
# OU
argocd app sync winenot

# 6. Monitorer les challenges
kubectl -n winenot get challenge -w
# Doit passer de "pending" -> "valid" en 30-60 secondes

# 7. Vérifier certificat READY
kubectl -n winenot get certificate -o wide
# lvp-ovh-tls et winenot-tls doivent afficher READY=True
```

**Commandes de diagnostic rapide:**
```bash
# Voir tous les issuers
kubectl get clusterissuer,issuer -A

# Voir challenges en cours
kubectl -n winenot get challenge,order -o wide

# Logs cert-manager détaillés
kubectl logs -n cert-manager -l app.kubernetes.io/name=cert-manager --tail=100 | grep -i 'http01\|acme\|propagation'

# Tester le chemin ACME manuellement
curl -v http://lvp.ovh/.well-known/acme-challenge/test-token
# Doit retourner 404 du solver cert-manager, PAS d'Apache

# Vérifier Traefik routes
kubectl -n kube-system logs -l app.kubernetes.io/name=traefik | grep -i 'acme\|http01' | tail -20
```

**Troubleshooting common:**

| Problème | Cause | Élix |
|----------|-------|------|
| Challenge `pending` > 10min | Traefik n'expose pas /.well-known/acme-challenge | Vérifier che Apache n'écoute pas port 80; vérifier solver ingress existe |
| `wrong status code 404` | Trafik route ACME paths au backend | Ajouter path explicit pour ACME dans ingress |
| Solver pods non créés | cert-manager pods down | kubectl -n cert-manager get pod; check logs |
| Certificate secret n'existe pas | Challenge jamais réussi | Voir statut du challenge: `kubectl -n winenot describe challenge <name>` |

# Info
url: lvp.ovh
server: ssh -p 7022 debian@lvp.ovh
email: lvp.communautee@gmail.com

Verifie que tu est toujours dans WSL sinon execute une fois 'wsl'

---
> Source: [Malaudos35/WineNot](https://github.com/Malaudos35/WineNot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
