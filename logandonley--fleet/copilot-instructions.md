## fleet

> Fleet is managed as a Nix flake and deployed with Colmena, so changes should always keep host reproducibility front of mind. Use this guide to stay consistent with the existing configuration style.

# Repository Guidelines

Fleet is managed as a Nix flake and deployed with Colmena, so changes should always keep host reproducibility front of mind. Use this guide to stay consistent with the existing configuration style.

## Project Structure & Module Organization
- `flake.nix` wires external inputs and exposes the Colmena hive plus the development shell.
- `hosts.nix` is the single inventory; per-host configs live in `hosts/<name>/` alongside hardware profiles and shared logic in `hosts/common.nix`.
- Service modules are grouped under `modules/<domain>/` and exposed via the `fleet.<domain>.<service>` option namespace (e.g. `modules/dev/gitea.nix`).
- Keep assets such as TLS material inside the relevant module options (`fleet.security.*`) rather than committing secrets.

## Build, Test, and Development Commands
- `nix develop` brings up the shell with Colmena installed; run it before modifying modules.
- `nix flake check` ensures all modules evaluate and option contracts stay valid.
- `colmena apply --on alpha --dry-run switch` validates the `alpha` host without touching state; drop `--dry-run` to deploy.
- `colmena apply --on bravo switch` publishes Git services; prefer tagging multiple hosts via the `hosts.nix` tags when coordinating changes.

## Coding Style & Naming Conventions
- Use two-space indentation and keep attribute sets alphabetised where practical.
- Follow the existing section banners (`# ===`) and organise modules as `options` then `config`.
- Name options `fleet.<domain>.<service>` and expose user-tunable settings through `mkOption` with informative defaults.
- Prefer descriptive attribute keys (`domain`, `port`, `description`) and consistent casing for hostnames and routes.

## Testing Guidelines
- Extend or adjust modules under `modules/` and rerun `nix flake check` before review.
- For behavioural changes, exercise `colmena apply --on <host> --dry-run switch` to confirm activations succeed.
- Capture any manual verification (e.g. Grafana reachable on port 3000) in the pull request notes so reviewers can mirror the checks.

## Commit & Pull Request Guidelines
- Match the existing history of short, lower-case imperative summaries (e.g. `adding reverse proxy`).
- Each PR should explain the motivation, list affected hosts or services, and call out required secrets or DNS updates.
- Include screenshots or command transcripts when modifying user-facing dashboards or endpoints.
- Reference related issues or host tags for traceability, and ensure CI/CD or deployment checks are linked before requesting review.

---
> Source: [logandonley/fleet](https://github.com/logandonley/fleet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
