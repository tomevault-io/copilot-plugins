## slurm-plugin

> This repository is a single **Maven-based Jenkins plugin** (the Slurm plugin). There is no database, frontend, or `docker-compose` — building, testing, and running the dev instance only need JDK + Maven + Python.

# AGENTS.md

## Cursor Cloud specific instructions

This repository is a single **Maven-based Jenkins plugin** (the Slurm plugin). There is no database, frontend, or `docker-compose` — building, testing, and running the dev instance only need JDK + Maven + Python.

### Toolchain notes (already provisioned on the VM)
- **JDK 21** and **Maven 3.9.x** are used. The Jenkins parent POM (`plugin:5.21`) requires **Maven >= 3.9**; the distro `apt` Maven (3.8.x) is too old, so a 3.9.x is installed at `/opt/maven` and wired via `update-alternatives` (`mvn` resolves to it).
- The build's `generate-sources` phase invokes `python` (not `python3`) via `exec-maven-plugin` to run `scripts/preprocess-openapi-spec.py`. `python-is-python3` is installed so `python` resolves to Python 3. Without a `python` on PATH, `mvn compile/test/package` fails early.
- The build auto-generates a Slurm REST API client from `src/main/resources/openapi/slurm-v0.0.42.json` into `target/generated-sources/`. A `mvn clean` wipes it; it is regenerated on the next build.

### Build / test / lint / run (standard commands; see README.md and CLAUDE.md for the full list)
- Build the plugin: `mvn -ntp clean package` → produces `target/slurm.hpi` (also runs the 43 unit tests).
- Tests only: `mvn -ntp test` (JUnit 5 only — JUnit 4 imports are banned by the `ban-junit4-imports` enforcer rule).
- Run dev instance: `mvn -ntp hpi:run`, then open `http://localhost:8080/jenkins`. Dev mode starts in `DEVELOPMENT` install state with **no security realm**, so no login is required. A CSRF crumb IS still enforced — fetch it from `/jenkins/crumbIssuer/api/json` and send it as the `Jenkins-Crumb` header (with the matching session cookie) for any POST (e.g. the `/jenkins/scriptText` Script Console). `hpi:run` is long-running; start it in a background/tmux session.

### Lint caveat (pre-existing)
- `mvn spotless:check` currently **FAILS on a clean checkout** because committed sources contain trailing whitespace. This is a pre-existing repo state, not an environment problem. Do not mass-reformat unrelated files to "fix" it; run `mvn spotless:apply` only on files you intentionally change.

### Slurm cluster is NOT required for local dev
- Building, unit tests, `hpi:run`, and configuring a Slurm cloud all work **without** a real Slurm cluster. A live `slurmrestd` (Slurm 24.11+ with JWT) is only needed to actually provision agents end to end. You can create and persist a `SlurmCloud` + `SlurmJobTemplate` (via UI, JCasC, or the Script Console) without any cluster.

### End-to-end tests (local Jenkins + live Slurm)
- Scripts live under `scripts/e2e/`. Copy `config.env.example` → `config.env` (gitignored) and set **your** SSH host, cloud name, and job paths — nothing environment-specific is hardcoded in committed scripts.
- Start `mvn hpi:run`, then the WSL agent tunnel (`start-tunnel.sh`), then `powershell -File scripts/e2e/run-e2e.ps1`.
- Agent tunnel must run in WSL if endpoint security blocks Windows OpenSSH `-R` forwards. Jenkins UI stays on `localhost:8080`; agents use the tunneled URL configured in your Slurm cloud (typically `http://localhost:5000/jenkins/` on the controller).
- Use a writable `current_working_directory` on compute nodes (e.g. `/tmp/jenkins`), not a restricted home directory owned by another service account.

### Dedicated Jenkins instance (Docker + live Slurm)
- Scripts live under `scripts/jenkins-instance/`. Copy `config.env.example` → `config.env` (gitignored).
- `deploy.ps1` — Docker Jenkins LTS on remote host, install `target/slurm.hpi`.
- `configure.ps1` — Slurm cloud, JWT credential, e2e jobs (direct agent URL, no tunnel).
- `run-e2e.ps1` — build, deploy, and run smoke jobs. See `scripts/jenkins-instance/README.md`.

---
> Source: [naveenrajm7/slurm-plugin](https://github.com/naveenrajm7/slurm-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
