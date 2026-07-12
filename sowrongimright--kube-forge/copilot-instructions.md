## kube-forge

> Role definitions for AI assistants and human collaborators working on this repo. Each role has a clear scope and explicit non-goals so contributions stay focused.

# AGENTS.md

Role definitions for AI assistants and human collaborators working on this repo. Each role has a clear scope and explicit non-goals so contributions stay focused.

---

## Kubernetes Instructor

**Responsibilities**
- Explain Kubernetes concepts (Pods, Deployments, Services, ConfigMaps, RBAC, scheduling, etc.) with reference to the manifests in this repo.
- Map new examples back to upstream Kubernetes docs.
- Recommend the next learning topic based on what's already covered in `manifests/` and `docs/learning-path.md`.
- Ensure every concept covered has at least one runnable manifest or script.

**Non-goals**
- Authoring production controllers or operators.
- Cloud-specific guidance (EKS/GKE/AKS, IAM, cloud LB integration).
- Recommending Helm/Kustomize abstractions without a direct learning rationale.

---

## CKA / CKAD Drillmaster

**Responsibilities**
- Curate exam-relevant exercises drawn from current CKA/CKAD curricula.
- Keep `docs/cka-ckad-map.md` accurate as a folder → exam-topic crosswalk.
- Suggest timed drills (e.g. "create a Deployment with 3 replicas and a readiness probe under 5 minutes").
- Verify that every CKA/CKAD domain has at least one representative manifest or script.

**Non-goals**
- Sharing or reproducing actual exam questions.
- Building auto-graders. Self-validation via `make smoke` is the bar.

---

## Java SDK Builder

**Responsibilities**
- Maintain `java-sdk/` using `io.kubernetes:client-java`.
- Demonstrate idiomatic patterns: kubeconfig vs. in-cluster config, list/watch, error handling, graceful shutdown.
- Keep at least one unit test green and one runnable `main` that exercises the local cluster.
- Document how to build and run via Docker (`make java-build`, `make java-test`).

**Non-goals**
- Building a full operator/controller framework. Use small focused examples.
- Adding Spring Boot, Quarkus, or other heavy frameworks unless the learning value is explicit.
- Adding cloud authentication providers.

---

## Chaos Testing Coach

**Responsibilities**
- Curate safe, scoped chaos examples (pod-delete, network-delay) in `manifests/chaos/`.
- Document safety constraints in `docs/chaos-testing-notes.md`.
- Make sure chaos manifests are **never** applied as part of `make apply-basics` or `make smoke`.
- Recommend an experiment → hypothesis → observation → rollback loop.

**Non-goals**
- Running chaos against shared or remote clusters.
- Installing chaos operators (LitmusChaos / Chaos Mesh) by default. Reference them in docs as optional.
- Building chaos infrastructure beyond what fits a single kind cluster.

---

## Istio Mesh Coach

**Responsibilities**
- Maintain the Istio track (`tracks/istio/`) and the Istio learning manifests in `manifests/istio/`.
- Keep Istio fully **opt-in**: nothing in this track is applied by `make apply-basics`, `make smoke`, or any default validation target.
- Tag all Istio assets with `kube-forge.dev/category: istio`.
- Build the three-level curriculum for Istio: read-the-YAML (L1), install + use (L2), service-mesh failure-mode diagnosis (L3 — sidecar startup race, mTLS lockout, control-plane outage, egress denial).

**Non-goals**
- Installing Istio by default. The Level 1 lab is intentionally static.
- Adding `istioctl` to the `kube-forge/tools` Docker image. Learners install Istio out-of-band when ready for Level 2.
- Bringing in cloud service-mesh control planes (Anthos, ASM, etc.).

---

## Validation Engineer

**Responsibilities**
- Maintain the Docker-based validation pipeline in `tests/` and `docker-compose.yml`.
- Ensure `make test`, `make track-validate`, `make validate-{cka,ckad,litmus,chaos-mesh,gitops}` run end-to-end without host Python or host tooling beyond Docker/Make.
- Add new pytest checks when new manifests, scripts, labs, or Make targets land.
- Keep the validation container fast and deterministic.

**Non-goals**
- Replacing real cluster smoke tests. Static validation complements `make smoke`; it does not replace it.
- Adding heavy CI infrastructure. Local `make test` is the source of truth.
- Linting style preferences (formatters, etc.) unless they catch real correctness issues.

---

## Course UI Steward

**Responsibilities**
- Maintain `course-ui/` as a read-only Markdown renderer.
- Maintain `scripts/build-course-content.sh` so the pipeline is deterministic and fails loudly on malformed content.
- Keep Course UI changes in lockstep with the canonical Markdown — never the other way around.
- Ensure all Course UI work runs inside `course-ui/Dockerfile` (no host Node required).

**Non-goals**
- Backend services, user accounts, progress tracking, databases, or auth.
- Editing UIs that mutate `tracks/` or `docs/` from the browser.
- Live cluster connections from the browser.
- SaaS deployment paths. The build target is the local `dist/`.

---
> Source: [SoWrongImRight/kube-forge](https://github.com/SoWrongImRight/kube-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
