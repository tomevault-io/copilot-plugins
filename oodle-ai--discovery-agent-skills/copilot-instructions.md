## discovery-agent-skills

> Discover a company's tech stack, observability stack, infrastructure scale, observability costs, and pain points across all environments. Detection is agent-driven; all volume and cost figures are computed by deterministic collector scripts with raw API evidence, merged into a single verifiable executive-level HTML report.


# Infrastructure & Observability Discovery

This skill guides the agent through a systematic discovery of a company's tech stack, observability stack, infrastructure scale, observability costs, and operational pain points. The output is one self-contained HTML report: an executive summary up top (scannable in under 2 minutes), collapsible per-tool deep dives below, and a provenance appendix linking every measured figure to the raw API response that produced it.

For install instructions, see [README.md](README.md).

## Division of labor (the core rule)

```
You (the agent)                      Collector scripts (deterministic)
────────────────────────             ──────────────────────────────────
detect environments & tools     →    collectors/<tool>/collect.py
ask qualitative questions       →    compute every volume/cost figure
write context.json              →    write summary.json + evidence/
run the report generator        →    report/generate_report.py writes the HTML
```

**You never compute a reported figure.** No improvised PromQL, no unit conversions, no arithmetic on API responses, no editing the report HTML. Every number in the report comes from a collector's `summary.json`; everything you contribute goes into `context.json` and is rendered as qualitative or "reported — not verified". If a collector fails, the answer is a documented gap — never a substituted estimate.

## Principles

1. **Non-destructive only.** Never modify, delete, or write to any system. All operations are read-only.
2. **Deterministic figures.** Volume and cost numbers come only from collector scripts, which capture the raw API responses they used (evidence files).
3. **Observability costs only.** Discover observability spend (CloudWatch, Datadog, self-hosted stack footprint, licenses). Never query or report account-wide cloud costs. For everything that isn't observability, record broad inventory only (counts, instance families, telemetry-relevant managed services).
4. **Never silently omit.** Anything that couldn't be collected appears in the report's Coverage & Gaps section with the reason and a remediation.
5. **Rate-limited.** Wait 1-2 seconds between ad-hoc API calls; collectors throttle themselves.
6. **Ask when uncertain.** Information that can't be discovered programmatically is asked of the user and recorded as user-reported.
7. **Plan first.** Always present the discovery plan and get user approval before executing.
8. **Multi-environment aware.** Discover all environments (dev, staging, prod, etc.) separately.

## Execution Flow

### Phase 0: Present Plan

Before doing anything, present this plan to the user and ask for approval:

```
I'll discover your infrastructure and observability setup. Here's my plan:

1. **Environment Detection** — Identify all environments (cloud accounts, k8s clusters, regions)
2. **Tech Stack Discovery** — Languages, frameworks, databases, message queues (broad inventory)
3. **Infrastructure Inventory** — Compute scale and telemetry-relevant managed services (counts only)
4. **Observability Stack Discovery** — Monitoring, logging, tracing, alerting tools
5. **Scale & Cost Measurement** — For each detected observability tool I'll run a read-only
   collector script that queries its usage/billing APIs and saves the raw responses as evidence.
   Costs are scoped to observability spend only — I will not query your overall cloud bill.
6. **Pain Points** — Observability-specific: alert fatigue, coverage gaps, tool sprawl, cost
7. **Report** — A single HTML report; every measured number links to its raw API evidence,
   and anything that couldn't be collected is listed with the reason.

All operations are read-only. Collector scripts run locally via `uv run`; nothing is uploaded.
Shall I proceed?
```

Wait for user confirmation before continuing.

### Phase 0.5: Prerequisites

Collector scripts are PEP 723 Python scripts run with `uv`:

```bash
uv --version 2>/dev/null
python3 --version 2>/dev/null   # needs >= 3.11
```

- If `uv` is missing, offer to install it (`curl -LsSf https://astral.sh/uv/install.sh | sh`) or via `pipx install uv` / `brew install uv`. Ask before installing anything.
- If the user declines or the machine is airgapped, continue in **degraded mode**: skip all collectors, record each skipped one in `context.json.skipped_collectors`, and still generate the report — the volumes/costs will appear as gaps.

### Phase 1: Environment Detection

Discover all environments by checking these sources in order. Stop each check after 5 seconds if no response.

**Before starting Phase 1, cache commonly used Kubernetes data to avoid redundant API calls:**

```bash
kubectl get pods -A -o json 2>/dev/null > /tmp/discovery-pods.json
kubectl get crd 2>/dev/null > /tmp/discovery-crds.txt
```

Use `/tmp/discovery-pods.json` and `/tmp/discovery-crds.txt` for all subsequent pod and CRD queries instead of re-fetching from the API.

#### 1.1 Kubernetes Clusters

```bash
kubectl config get-contexts 2>/dev/null
kubectl cluster-info 2>/dev/null
kubectl get namespaces -o json 2>/dev/null | jq -r '.items[].metadata.name'
```

#### 1.2 Cloud Provider Detection

```bash
# AWS
aws sts get-caller-identity 2>/dev/null
aws ec2 describe-regions --query 'Regions[].RegionName' -o text 2>/dev/null

# GCP
gcloud config list --format=json 2>/dev/null
gcloud projects list --format=json 2>/dev/null

# Azure
az account list -o json 2>/dev/null
```

#### 1.3 Infrastructure-as-Code Detection

```bash
find . -maxdepth 4 -name "*.tf" -o -name "*.tfvars" | head -20
find . -maxdepth 4 -name "pulumi.*" -o -name "Pulumi.yaml" | head -10
find . -maxdepth 4 -name "cdk.json" -o -name "serverless.yml" | head -10
find . -maxdepth 4 -name "docker-compose*.yml" -o -name "docker-compose*.yaml" | head -10
find . -maxdepth 4 -name "helmfile.yaml" -o -name "Chart.yaml" | head -20
```

#### 1.4 CI/CD Detection

```bash
ls -la .github/workflows/ 2>/dev/null
ls -la .gitlab-ci.yml 2>/dev/null
ls -la Jenkinsfile 2>/dev/null
ls -la .circleci/ 2>/dev/null
ls -la .buildkite/ 2>/dev/null
```

#### 1.5 Docker & Non-Kubernetes Environments

```bash
docker compose config --services 2>/dev/null || docker-compose config --services 2>/dev/null
docker ps --format '{{.Names}}\t{{.Image}}\t{{.Status}}' 2>/dev/null | head -20
systemctl list-units --type=service --state=running 2>/dev/null | grep -i "postgres\|mysql\|mongo\|redis\|nginx\|haproxy\|kafka\|rabbit\|elastic\|prometheus\|grafana\|docker" | head -20
```

#### 1.6 Helm Releases & CRDs

```bash
helm list -A -o json 2>/dev/null | jq -r '.[] | "\(.namespace)\t\(.name)\t\(.chart)"' | head -30
grep -i "monitoring\|observability\|prometheus\|datadog\|elastic\|cert-manager\|istio\|linkerd" /tmp/discovery-crds.txt | head -20
```

If environments cannot be determined from the above, ask the user:
```
I couldn't automatically detect all your environments. Could you tell me:
- What cloud provider(s) do you use?
- How many environments do you have (dev/staging/prod)?
- Do you use Kubernetes? If so, how many clusters?
```

### Phase 2: Tech Stack Discovery

Broad inventory only — names and counts, suitable for tags in the report.

```bash
# Languages & frameworks from package files
find . -maxdepth 4 \( -name "package.json" -o -name "go.mod" -o -name "requirements.txt" \
  -o -name "Pipfile" -o -name "pyproject.toml" -o -name "Gemfile" -o -name "pom.xml" \
  -o -name "build.gradle" -o -name "Cargo.toml" -o -name "mix.exs" \
  -o -name "*.csproj" -o -name "composer.json" \) 2>/dev/null | head -30

# Databases / caches / queues running in Kubernetes
kubectl get services -A -o json 2>/dev/null | jq -r '.items[] | select(.metadata.name | test("postgres|mysql|mongo|redis|elastic|kafka|rabbit|cassandra|dynamo|cockroach|clickhouse")) | "\(.metadata.namespace)/\(.metadata.name)"'

# Telemetry-relevant managed services (counts only — these are integration
# candidates for Oodle onboarding)
aws rds describe-db-instances --query 'length(DBInstances)' 2>/dev/null
aws elasticache describe-cache-clusters --query 'length(CacheClusters)' 2>/dev/null
aws es list-domain-names 2>/dev/null | jq '.DomainNames | length'
aws eks list-clusters -o json 2>/dev/null | jq '.clusters | length'
aws kafka list-clusters --query 'length(ClusterInfoList)' 2>/dev/null
aws lambda list-functions --query 'length(Functions)' 2>/dev/null
aws sqs list-queues 2>/dev/null | jq '.QueueUrls | length'
gcloud container clusters list --format=json 2>/dev/null | jq 'length'
```

### Phase 3: Infrastructure Inventory

Aggregates only: total nodes, vCPU, memory, instance-type families per environment. Do NOT collect per-resource detail, individual node IPs, or anything cost-related — this phase exists to size the environment for onboarding, not to audit it.

If any kubectl context exists, run the k8s inventory collector — do not sum node capacities yourself:

```bash
uv run collectors/k8s/collect.py --all-contexts --output-dir ./discovery-output/k8s
# or, for selected clusters:
uv run collectors/k8s/collect.py --context prod-eks --context stg-eks \
  --output-dir ./discovery-output/k8s
```

It measures nodes, vCPU, memory (GiB), and deployment counts per context with kubectl evidence, and its summary.json feeds the report's Compute/Services cards directly. For monitored-host counts, vendor collectors also contribute (e.g. the Datadog collector's `hosts.count` from the hosts API — preferred on Datadog-centric environments since it covers non-k8s hosts too).

Environment *naming/classification* (which cluster is prod vs staging) is yours: record it in `context.json.environments[]`, using the collector's per-context inventory for the numbers.

### Phase 4: Observability Stack Discovery

#### 4.1 Detection

```bash
# Monitoring/metrics pods (from cached pod data)
jq -r '.items[] | select(.metadata.name | test("prometheus|thanos|mimir|cortex|victoriametrics|vmagent|datadog|newrelic|nri-|dynatrace|oneagent|splunk|grafana|honeycomb|lightstep")) | "\(.metadata.namespace)/\(.metadata.name)"' /tmp/discovery-pods.json | head -20

# Agents as DaemonSets
kubectl get daemonsets -A -o json 2>/dev/null | jq -r '.items[] | select(.metadata.name | test("datadog|newrelic|dynatrace|oneagent|splunk|fluentd|fluent-bit|node-exporter|filebeat")) | "\(.metadata.namespace)/\(.metadata.name)"'

# Helm-based detection (catches operator-managed stacks)
helm list -A -o json 2>/dev/null | jq -r '.[] | select(.chart | test("prometheus|datadog|newrelic|dynatrace|grafana|loki|tempo|splunk|elastic|victoria|mimir")) | "\(.namespace)\t\(.name)\t\(.chart)"'

# CRD-based detection (from cached data)
grep -i "monitoring.coreos.com\|datadoghq.com\|newrelic.com\|dynatrace.com\|opentelemetry" /tmp/discovery-crds.txt | head -10

# Logging stacks (from cached pod data)
jq -r '.items[] | select(.metadata.name | test("fluentd|fluent-bit|logstash|loki|vector|filebeat|elasticsearch|opensearch|kibana")) | "\(.metadata.namespace)/\(.metadata.name)"' /tmp/discovery-pods.json | head -10

# Tracing (from cached pod data)
jq -r '.items[] | select(.metadata.name | test("jaeger|zipkin|tempo|otel|opentelemetry|honeycomb|lightstep")) | "\(.metadata.namespace)/\(.metadata.name)"' /tmp/discovery-pods.json | head -10

# Alerting & on-call
jq -r '.items[] | select(.metadata.name | test("alertmanager")) | "\(.metadata.namespace)/\(.metadata.name)"' /tmp/discovery-pods.json | head -5
kubectl get configmaps -A -o json 2>/dev/null | jq -r '.items[] | select(.data | tostring | test("pagerduty|opsgenie|victorops|incident")) | "\(.metadata.namespace)/\(.metadata.name)"' | head -5
```

#### 4.2 Collector routing table

For each detection below, plan to run the matching collector in Phase 5. A tool can match more than one row (e.g., Datadog SaaS + self-hosted Prometheus).

| Detected | Collector |
|---|---|
| any kubectl context | `collectors/k8s/collect.py` (inventory; run in Phase 3) |
| prometheus / thanos / victoriametrics / vmagent pods, `monitoring.coreos.com` CRDs, loki / tempo pods | `collectors/promstack/collect.py` *(planned — record as skipped until shipped)* |
| mimir / cortex pods or charts | `collectors/mimir/collect.py` *(planned)* |
| elasticsearch / kibana pods, AWS ES domains | `collectors/elasticsearch/collect.py` *(planned)* |
| opensearch pods, AWS OpenSearch/AOSS domains | `collectors/opensearch/collect.py` *(planned)* |
| datadog agent/operator pods, DD CRDs, or the user says they use Datadog | `collectors/datadog/collect.py` |
| `aws` CLI authenticated (CloudWatch is in use by default on AWS) | `collectors/cloudwatch/collect.py` *(planned)* |
| `gcloud` CLI authenticated | `collectors/gcp/collect.py` *(planned)* |

For rows marked *(planned)*, the collector does not exist yet in this version of the skill: record the tool in `context.json.skipped_collectors` with reason "collector not yet available", and ask the user for representative numbers instead (recorded as `user_reported`).

### Phase 5: Scale & Cost Measurement (collectors)

#### 5.1 Infrastructure aggregates

Measured by the k8s collector in Phase 3 (`./discovery-output/k8s/summary.json`). Your job here is only classification: copy the per-context numbers from its inventory into `context.json.environments[]` with the right env labels (prod/staging/dev). Do not recompute totals — the report's cards read the collector figures directly.

#### 5.2 Observability volumes & costs (collectors only)

For each collector selected in Phase 4.2:

1. Ask the user for the credentials it needs (e.g., Datadog API + application key) and pass them via environment variables — never write credentials to files.
2. Run it with a per-tool output directory:

```bash
mkdir -p ./discovery-output
DD_API_KEY=... DD_APP_KEY=... uv run collectors/datadog/collect.py \
  --site us1 --lookback 30d --output-dir ./discovery-output/datadog
```

3. Read the collector's stdout. It ends with a figures/gaps count. Do not parse evidence files or recompute anything from them.
4. If a collector exits non-zero or errors: capture stderr, retry once, and if it still fails record it in `context.json.skipped_collectors` with the error as the reason. **Never substitute your own estimate for a failed collector.**

Hard rules for this phase:

- Do not write your own queries against observability backends (no PromQL, no `_cat` APIs, no usage API calls). The collectors own those.
- Do not port-forward to in-cluster services yourself; collectors that need it do it internally with proper cleanup.
- If the user volunteers numbers ("we do about 200GB of logs a day"), record them in `context.json.user_reported` — they never become figures.

#### 5.3 If no collector covers a detected tool

Ask the user for representative numbers (last 7–30 days), and record their answers verbatim in `context.json.user_reported`:

```
I need a few numbers for <tool>, which I can't measure automatically. Approximations are fine:
1. Active time series / metrics ingestion rate?
2. Log volume (GB/day)?
3. Trace volume (GB/day or spans/sec)?
4. Data retention for metrics / logs / traces?
```

### Phase 6: Observability Cost Discovery

**Scope: observability spend only. Never query, estimate, or report account-wide cloud costs.**

- Measured observability spend comes from collectors (Datadog estimated cost; CloudWatch via Cost Explorer scoped to `SERVICE=AmazonCloudWatch` once that collector ships; GCP Monitoring/Logging once that collector ships).
- For vendors without a collector, ask the user and record as `user_reported` with `area: "cost"`:

```
A couple of cost questions, scoped to observability only:
- What do you pay monthly/annually for observability vendors (Datadog, Splunk, New Relic, Grafana Cloud, ...)?
- Any committed contracts or credits worth noting?
```

Do not ask about compute, database, or total cloud spend.

### Phase 7: Pain Points Discovery

Ask the user targeted questions about **observability-specific** pain points only. Do NOT ask about general infrastructure, deployment, or application-level pain points.

```
Based on what I've found, I have a few questions about observability pain points:

1. **Alert fatigue** — How many alerts do you receive per day/week? Are most actionable?
2. **Observability gaps** — Are there services or systems with insufficient monitoring, logging, or tracing?
3. **Troubleshooting time** — How long does it typically take to identify root cause of incidents?
4. **Observability cost** — Are observability costs growing faster than your infrastructure? Vendor lock-in concerns?
5. **Tool sprawl** — Do you switch between too many observability tools during incidents?
6. **Data retention** — Are you satisfied with retention for metrics/logs/traces? Compliance requirements unmet?
7. **Correlation gaps** — Can you easily correlate metrics, logs, and traces for a single request?
```

Keep the top 3-5 for the report.

### Phase 8: Generate the Report

#### 8.1 Write context.json

Write `./discovery-output/context.json` with everything qualitative you discovered. It must validate against `schemas/context.schema.json`. Shape:

```json
{
  "schema_version": "1.0",
  "company": "<company name if known>",
  "generated_by": "<your platform, e.g. claude-code>",
  "team_size": "<user-reported or null>",
  "environments": [
    {"name": "production", "kind": "prod", "cloud": "aws", "region": "us-east-1",
     "cluster": "prod-eks", "nodes": 24, "services": 31, "vcpu": 192, "memory_gi": 768}
  ],
  "tech_stack": {"languages": ["Go"], "databases": ["PostgreSQL"], "messaging": ["Kafka"],
                 "infra": ["Terraform", "Helm"], "other": []},
  "infra_inventory": {"kubernetes_clusters": 2, "total_nodes": 30, "total_services": 59,
                      "instance_families": ["m6i"], "regions": ["us-east-1"],
                      "managed_services": [{"type": "RDS", "count": 4}]},
  "observability_stack": [
    {"signal": "metrics", "tools": ["Datadog"]},
    {"signal": "logs", "tools": ["Datadog Logs"]},
    {"signal": "alerting", "tools": ["Datadog Monitors", "PagerDuty"]}
  ],
  "pain_points": [{"title": "...", "detail": "...", "recommendation": "..."}],
  "user_reported": [{"label": "Splunk contract", "value": "$200K/yr", "area": "cost"}],
  "skipped_collectors": [{"tool": "cloudwatch", "reason": "aws CLI not configured"}],
  "narrative": {"overview": "1-3 sentence executive overview."}
}
```

Notes:
- Numbers in `environments[]`/`infra_inventory` are inventory facts you gathered in Phases 1–3; they render as "reported".
- Keep narrative paragraphs short and factual; they are rendered verbatim.

#### 8.2 Run the generator

```bash
uv run report/generate_report.py \
  --context ./discovery-output/context.json \
  --summaries ./discovery-output/*/summary.json \
  --out ./discovery-report.html \
  --strict
```

If no collector ran (degraded mode), omit `--summaries` entirely — the generator still produces the report from `context.json`, with every skipped collector listed in Coverage & Gaps. Tell the user the report contains only reported (unverified) data in that case.

**Hard rules:**
- Figures come ONLY from `summary.json` files. Never edit `discovery-report.html`. Never transcribe, recompute, convert units of, or round any collector figure in anything you write.
- Do not regenerate or hand-write any HTML. If a section looks wrong, fix `context.json` or re-run a collector, then re-run the generator.

Open the report:

```bash
open ./discovery-report.html 2>/dev/null || xdg-open ./discovery-report.html 2>/dev/null || \
  echo "Report saved to: $(pwd)/discovery-report.html"
```

### Phase 8.5: Gap Review

The generator prints a coverage summary (figures collected per collector + every gap with remediation). Walk the user through it:

1. Summarize what was measured vs. what's missing.
2. For each gap with a remediation (e.g., "grant usage_read to the application key", "rerun with --loki-endpoint"), ask whether they want to fix and re-run.
3. After any re-run, regenerate the report (Phase 8.2) and re-open it.

End by telling the user: the `discovery-output/` directory contains the raw evidence for every figure; the report's Provenance Appendix maps each figure to its evidence file.

## Rate Limiting & Safety

| Target System | Max Rate | Backoff |
|---------------|----------|---------|
| Kubernetes API | 5 req/sec | 2s on 429 |
| AWS API | 2 req/sec | 5s on throttle |
| GCP API | 2 req/sec | 5s on throttle |
| Collector scripts | self-throttled (150ms between requests, circuit breaker) | built in |

### Safety Rules

- **Never** run `kubectl delete`, `kubectl apply`, `kubectl patch`, or any write operation.
- **Never** run cloud CLI commands that create, modify, or delete resources.
- **Never** modify files in the user's workspace except `./discovery-output/` and the report.
- **Never** write credentials to disk; pass them as environment variables to collectors.
- **Never** compute, convert, or round report figures yourself — that is the collectors' job.
- **If a command hangs** for more than 10 seconds, kill it and move on.
- **If a command requires credentials** you don't have, skip it and note the gap.

## Failure Handling

| Situation | Action |
|-----------|--------|
| CLI tool not installed | Note it as unavailable, skip related checks |
| `uv` unavailable and user declines install | Degraded mode: skip collectors, record in skipped_collectors |
| Collector exits non-zero | Capture stderr, retry once, then record in skipped_collectors |
| No credentials/permissions | Ask user once; otherwise record gap |
| Command times out | Kill after 10s, note as "unable to assess" |
| Multiple clusters/accounts | Discover each separately, label by environment |

## References

- [Oodle](https://oodle.ai) — Observability platform
- `schemas/summary.schema.json` — collector output contract
- `schemas/context.schema.json` — agent context contract
- `examples/sample-datadog-report.html` — reference output style

---
> Source: [oodle-ai/discovery-agent-skills](https://github.com/oodle-ai/discovery-agent-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
