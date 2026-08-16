## kubecost

> - When asked to review a PR, compare this current branch against the `develop` branch. `git diff develop`.

# Agents.md

## PR Reviews

- When asked to review a PR, compare this current branch against the `develop` branch. `git diff develop`.
- Document your PR review in a new markdown file
- Point out any critical architectural or templating bugs
- Validate that the changes are coherent and concise
- Validate that the changes are consistent with the project's coding standards
- Validate that the changes are backwards compatible
- Test all configuration changes via `helm template`. Document the BEFORE and AFTER. Validate the output is valid kubernetes YAML.

Testing the changes via `helm template`:

```sh
# Remove any existing subcharts
rm -rf ./kubecost/charts

# Get the updated subchart
helm repo add finops-agent https://kubecost.github.io/finops-agent-chart/
helm repo update finops-agent
helm dependency build ./kubecost

# Templating
helm template kubecost ./kubecost
```

---
> Source: [kubecost/kubecost](https://github.com/kubecost/kubecost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
