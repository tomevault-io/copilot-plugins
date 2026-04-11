## kubernetes-sandbox

> This repository is a MicroK8s-focused Kubernetes sandbox. The primary pattern is:

# AGENTS.md

## Purpose

This repository is a MicroK8s-focused Kubernetes sandbox. The primary pattern is:

- `infrastruture/` provisions cluster-wide foundations such as Gateway API, cert-manager, DNS automation, AWS credential propagation, and NFS-backed storage classes.
- App directories such as `n8n/`, `keyclock/`, and `grafana+loki/` own namespace-scoped workloads, PVCs, Services, and routes.
- `n8n/` and `grafana+loki/` should be treated as the reference examples for how new apps are added.

Keep changes aligned with the existing operational model instead of introducing a second deployment style.

## Repository Shape

- `infrastruture/`: shared cluster setup and shared gateways.
- `default/`: default-namespace examples and templates.
- `n8n/`, `keyclock/`, `grafana+loki/`: app-specific deployments.
- `test/`: ad hoc manifests used for manual cluster validation.
- `bin/`: cluster backup and restore utilities.

The `keyclock/` directory name is intentionally misspelled in the current repo. Preserve existing paths unless the task is an explicit cleanup.

## Standard Working Rules

- Prefer the existing shell scripts in `scripts/` as the entrypoint for deploying or destroying a component. Do not bypass them unless the task is specifically to redesign the workflow.
- Assume `microk8s kubectl`, `microk8s helm`, `microk8s helm3`, `envsubst`, `base64`, `docker`, and standard Unix shell tools are the expected toolchain.
- When adding a new component, follow the existing directory contract:
  - component-local `.env` for live values and secrets
  - component-local `.env.sample` with placeholder values only
  - `scripts/` for deploy, destroy, and optional manifest-preview helpers
  - `manifests/` for raw Kubernetes YAML
  - `values/` only when Helm is involved
- Keep `.env.sample` files synchronized with any new required variables.
- Never commit new real secrets into `.env.sample`, manifests, or values files.

## Shell Script and `.env` Conventions

This repo relies heavily on shell scripts plus component-local `.env` files.

- Deploy scripts typically resolve their own location with `BASH_SOURCE[0]`, then `source ../.env`.
- The script prepares derived variables before applying manifests.
- Secret values are commonly base64-encoded in the script and injected into Secret manifests through `envsubst`.
- Non-secret config is usually left in plain manifests, Deployment env vars, or Helm values.

Follow the existing pattern when adding or updating scripts:

1. Resolve `SCRIPT_DIR`.
2. `source` the component `.env`.
3. Export helper paths such as `manifests_dir`, `values_dir`, or `secrets_dir`.
4. Export any base64-encoded variables needed by Secret manifests.
5. Run `envsubst` only on manifests that are intended to be templated.
6. Apply manifests in dependency order: namespace, secrets/config, storage, workloads, services, routes.

Important: some manifests intentionally contain environment-variable syntax that must survive into the running container. Do not blindly run `envsubst` across every file.

Existing examples:

- `n8n/scripts/deploy-n8n.sh` encodes Postgres and Box credentials, then templates only selected manifests.
- `keyclock/scripts/deploy.sh` follows the same pattern for Postgres and Keycloak admin credentials.
- `grafana+loki/scripts/deploy.sh` mixes `kubectl apply`, `helm upgrade --install`, a component `.env`, and a local `secrets/` file.

## Manifest Templating Rules

Use `envsubst` selectively.

- Safe targets: Secret manifests, storage-class manifests, PVC manifests, and other YAML files that are explicitly parameterized with `${...}` placeholders intended for deploy time.
- Common no-template target: ConfigMaps that embed shell or SQL snippets which should evaluate environment variables at container runtime instead of deploy time.

The Postgres init ConfigMaps in `n8n/` and `keyclock/` are the clearest examples of this rule. Preserve that distinction when editing or adding manifests.

## Secrets and Sensitive Values

There are two patterns in the repo:

- Component `.env` files hold app-local values and secrets.
- Some workflows also use separate files such as `grafana+loki/secrets/sandbox.json` for large secret payloads.

Guidelines:

- Keep secrets out of committed manifests whenever possible.
- If a Secret manifest is used, have the script prepare the encoded values.
- If a workload needs a file-based secret, prefer `kubectl create secret generic --from-file=...` from the deploy script.
- Treat checked-in `.env` files as sensitive if they already exist. Do not echo or reproduce their contents in documentation or responses.

## Gateway and Endpoint Conventions

Ingress in this repo is based on Gateway API, not Ingress resources.

Shared gateways live in `infrastruture/create-gateways.yaml`:

- `koeppster-lan-gateway`: internal LAN-facing traffic, typically `*.k8s.koeppster.lan`
- `johnkoepp-com-gateway`: public traffic, with HTTP and HTTPS listeners for `*.johnkoepp.com`

Component routes should follow this model:

- Each app owns its own `HTTPRoute` manifest inside its namespace.
- LAN-only endpoints attach to `koeppster-lan-gateway`.
- Public endpoints attach to `johnkoepp-com-gateway`.
- Public routes usually come in pairs:
  - an HTTPS route that forwards to the Service
  - an HTTP route that only redirects to HTTPS

For public DNS automation, the redirect route is usually labeled:

- `koeppster.net/aws_common_name: <fqdn>`
- `koeppster.net/aws_status: waiting`

That label pair is how the Route53 updater discovers records to create or update.

When a proxied app needs forwarded-header handling, copy the Keycloak pattern rather than inventing a new one. `keyclock/manifests/keycloak-gateway.yaml` shows the current precedent for setting `X-Forwarded-*` headers.

## Namespace Conventions

- App namespaces are usually created by the app deploy script.
- Namespaces that should receive AWS ECR credentials are labeled `koeppster.net/aws_enabled: "true"`.
- Shared infrastructure objects live in the `infrastructure` namespace even though the directory is spelled `infrastruture/`.

Preserve those labels and namespace boundaries when moving resources around.

## NFS Storage Class and PVC Standard

NFS-backed storage is the default persistence model in this repo.

Cluster-level setup:

- `infrastruture/create-infrastructure.sh` installs the NFS CSI driver.
- Storage classes are then applied from manifests such as:
  - `infrastruture/create-storage-class.yaml` -> `kube-nfs`
  - `infrastruture/postgres-storage-class.yaml` -> `kube-postgres`
  - `grafana+loki/manifests/grafana-storage-class.yaml` -> `kube-grafana`

Standard storage-class pattern:

- `provisioner: nfs.csi.k8s.io`
- `parameters.server: ${nfs_server_ip}`
- a share-specific `parameters.share`
- `reclaimPolicy: Delete`
- `volumeBindingMode: Immediate`
- mount options including `hard` and `nfsvers=4.1`

Standard PVC guidance:

- Most app PVCs should reference an existing NFS storage class rather than define a PV directly.
- Use `kube-nfs` for general app data unless the app needs a dedicated NFS share.
- Use `kube-postgres` for Postgres stateful data, typically through `volumeClaimTemplates` on the StatefulSet.
- Use a dedicated storage class such as `kube-grafana` only when the app intentionally maps to a separate NFS export.

When adding a new NFS-backed workload:

1. Decide whether an existing storage class is sufficient.
2. If not, add a new storage-class manifest that follows the same NFS CSI shape and points at a distinct share path.
3. Reference that storage class from the app PVC or StatefulSet `volumeClaimTemplates`.
4. Keep access mode choices explicit:
   - `ReadWriteOnce` for most single-pod stateful services
   - `ReadWriteMany` only when the workload actually benefits from shared write access

Do not add hostPath storage for app data unless the task explicitly calls for a local-only exception.

## Helm Usage

Helm is used selectively, not universally.

- `grafana+loki/` is the model for Helm-based app deployment.
- When Helm is used, keep chart values in a component-local `values/` directory.
- If persistence is required, prefer pre-created PVCs plus `existingClaim` settings, matching the Grafana and Loki examples.

## Safe Change Strategy

When editing this repo:

- Prefer small, local changes that preserve the existing deploy flow.
- Update shell scripts, manifests, and `.env.sample` together when introducing new variables.
- Keep resource names, namespaces, and path conventions stable unless a full cleanup is explicitly requested.
- Verify whether a manifest is meant to be static YAML or deploy-time templated YAML before changing it.
- Be careful with existing filename typos and mismatches; several scripts depend on exact current names.

## Useful Validation Steps

Common validation is manual and script-driven:

- Run the relevant component deploy script.
- Inspect rendered output with the component `check-deploy.sh` helper when present.
- Check pods, services, PVCs, and `HTTPRoute` resources with `microk8s kubectl`.
- For infrastructure changes, validate the target StorageClass, Gateway, certificate, and namespace labels.

There is no strong automated test harness in this repo. Prefer concrete deploy-path validation over hypothetical refactors.

## Command Line Notes

- As this is s microk8s environement, to run `kubectl` run `microk8s kubectl`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/koeppj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/koeppj)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
