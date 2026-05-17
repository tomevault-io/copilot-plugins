## keda

> KEDA ScaledObject and ScaledJob generation rules — auth patterns, scaling safety, best practices


# KEDA Rules

Always generate ScaledObject and ScaledJob resources with all of the following. Missing any item is a Critical finding.

## Required fields on every ScaledObject

```yaml
spec:
  scaleTargetRef:
    apiVersion: apps/v1           # Always explicit — don't rely on the default
    kind: Deployment              # Or StatefulSet — be explicit
    name: <exact-deployment-name>

  minReplicaCount: 1              # Use 0 only when cold-start latency is acceptable and documented
  maxReplicaCount: <N>            # Always set — never omit the ceiling

  pollingInterval: 30             # 30s is a safe default; lower only for latency-sensitive queues
  cooldownPeriod: 300             # 5-minute cooldown prevents thrashing

  advanced:
    restoreToOriginalReplicaCount: true   # Always set — restores replicas on ScaledObject deletion
```

## Required on every trigger

```yaml
triggers:
  - type: <scaler>
    metadata:
      activationThreshold: "<N>"  # or activationQueueLength / activationLagThreshold
      # Prevents activation on noise/sparse events
    authenticationRef:
      name: <trigger-auth-name>   # Always use TriggerAuthentication — never inline credentials
```

## TriggerAuthentication: prefer Pod Identity

```yaml
# ✅ Best practice — no static credentials
spec:
  podIdentity:
    provider: aws                 # aws | azure | gcp | aws-eks

# ⚠️ Only when Pod Identity is unavailable — rotate keys; scope the Secret tightly
spec:
  secretTargetRef:
    - parameter: <param>
      name: <secret-name>
      key: <key>
```

## Scaler-specific minimum IAM permissions (least privilege)

| Scaler | Minimum permission |
|---|---|
| `aws-sqs-queue` | `sqs:GetQueueAttributes`, `sqs:GetQueueUrl` — NOT `sqs:ReceiveMessage` |
| `kafka` | consumer group describe + offset fetch — NOT produce |
| `azure-servicebus` | `Listen` on the specific queue or subscription only |
| `prometheus` | No auth for cluster-internal Prometheus |

## Cron scaler rules

```yaml
# Cron windows must not overlap.
# Always pair with a Prometheus/queue trigger as a safety net for unexpected spikes.
# Use idleReplicaCount or minReplicaCount >= 1 to keep a warm pod during off-hours.
# Always set timezone explicitly — never rely on UTC as an unstated assumption.
- type: cron
  metadata:
    timezone: Europe/Berlin       # IANA timezone — always explicit
    start: "0 8 * * 1-5"
    end: "0 20 * * 1-5"
    desiredReplicas: "10"
```

## ScaledJob required fields

```yaml
spec:
  jobTargetRef:
    template:
      spec:
        restartPolicy: Never      # Required — Jobs must not use OnFailure with KEDA
        activeDeadlineSeconds: 3600  # Required — prevents zombie jobs
        containers:
          - resources:
              requests: { cpu: "500m", memory: "512Mi" }
              limits: { memory: "2Gi" }   # memory limit required; omit cpu limit
```

## Never generate

- Static credentials inlined directly in `ScaledObject` metadata — always use `TriggerAuthentication`
- `SQS` scaler with `sqs:ReceiveMessage` — KEDA only reads depth, it does not consume
- A separate `HorizontalPodAutoscaler` targeting the same Deployment — KEDA manages its own HPA
- `minReplicaCount: 0` without a comment explaining cold-start acceptance
- Overlapping cron windows — they produce undefined behavior
- `pollingInterval < 10` on SQS — each poll is a billable AWS API call
- Kafka scaler without `lagThreshold` set — defaults are rarely appropriate

## HPA conflict check

Before generating a ScaledObject, note if an HPA already targets the same Deployment. If it does, include:

```bash
# Delete existing HPA before applying ScaledObject
kubectl delete hpa <existing-hpa> -n <namespace>
```

---
> Source: [nitinjain999/platform-skills](https://github.com/nitinjain999/platform-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
