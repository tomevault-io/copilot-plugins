## gcp-log-pipeline

> A log ingestion and observability pipeline on GCP. Python producer → Kafka → Python consumer → Elasticsearch → Kibana. Infrastructure provisioned with Terraform, configured with Ansible.

# gcp-log-pipeline

A log ingestion and observability pipeline on GCP. Python producer → Kafka → Python consumer → Elasticsearch → Kibana. Infrastructure provisioned with Terraform, configured with Ansible.

## Repository map

```
terraform/      GCP infra: VPC, firewall, GCE instances, remote state in GCS
ansible/        Instance config and service deployment
  roles/
    docker/     Installs Docker on all hosts
    kafka/      Deploys Kafka + Zookeeper via Docker Compose
    elk/        Deploys ES + Kibana + consumer via Docker Compose
producer/       Python app that generates fake structured log events → Kafka
consumer/       Python app that reads from Kafka → indexes into Elasticsearch
Makefile        All common operations, see below
DECISIONS.md    Intentional tradeoffs — read before suggesting changes
```

## Key commands

```bash
make init          # terraform init
make plan          # terraform plan
make apply         # terraform apply — creates GCE instances
make destroy       # terraform destroy — always run when done to avoid charges
make inventory     # generates ansible/inventory.ini from terraform outputs
make deploy        # runs full ansible playbook (all roles)
make deploy-kafka  # runs kafka role only
make deploy-elk    # runs elk role only
make produce       # runs producer locally pointed at Kafka public IP
```

For staged debugging:
```bash
cd ansible
ansible-playbook -i inventory.ini playbook.yml --tags docker
ansible-playbook -i inventory.ini playbook.yml --tags kafka
ansible-playbook -i inventory.ini playbook.yml --tags elk
```

## Infrastructure

| Resource | Value |
|---|---|
| GCP project | `gcp-log-pipeline` |
| Region / zone | `europe-west1` / `europe-west1-b` |
| Kafka instance | `kafka-instance` · e2-medium · 20 GB · tag: `kafka` |
| ELK instance | `elk-instance` · e2-medium · 30 GB · tag: `elk` |
| VPC | `pipeline-vpc` · `10.0.1.0/24` |
| State bucket | `gcp-log-pipeline-tfstate-lausbelphegor` |
| SSH user | `debian` |
| SSH key | `~/.ssh/id_ed25519` (injected via instance metadata) |

## Firewall rules

| Rule | Ports | Source |
|---|---|---|
| `allow-ssh` | 22 | `var.your_ip` |
| `allow-kafka-external` | 9092 | `var.your_ip` (producer needs this) |
| `allow-kibana` | 5601 | `var.your_ip` |
| `allow-internal` | 9092, 9200, 2181, 29092 | `10.0.1.0/24` (VPC only) |

## Kafka listener model

The broker runs two listeners:
- `EXTERNAL://0.0.0.0:9092` — advertised as `<kafka_public_ip>:9092` — used by the local producer
- `INTERNAL://0.0.0.0:29092` — advertised as `kafka:29092` — used by the consumer inside the ELK compose network

Both `KAFKA_LISTENERS` and `KAFKA_ADVERTISED_LISTENERS` must be set explicitly. Do not remove or merge them — the dual-listener config is required because the producer runs outside GCP and the consumer runs inside the same Docker network as ES.

## Service ports

| Service | Host | Port | Reachable from |
|---|---|---|---|
| Zookeeper | kafka-instance | 2181 | VPC only |
| Kafka | kafka-instance | 9092 | your IP + VPC |
| Kafka internal | kafka-instance | 29092 | Docker network only |
| Elasticsearch | elk-instance | 9200 | VPC only |
| Kibana | elk-instance | 5601 | your IP only |

## Environment variables

Producer (set locally or via `make produce`):
```
KAFKA_BOOTSTRAP   Kafka public IP:9092   default: localhost:9092
KAFKA_TOPIC       Topic name             default: app-logs
```

Consumer (set in Docker Compose env, templated by Ansible):
```
KAFKA_BOOTSTRAP   Kafka private IP:9092
ES_HOST           http://elasticsearch:9200
KAFKA_TOPIC       app-logs
```

## Secrets and credentials

- `terraform/credentials.json` — GCP service account key for Terraform. Never commit. Never read or print the contents of this file.
- `terraform/terraform.tfvars` — contains `project_id` and `your_ip`. Never commit.
- Both are in `.gitignore`. If asked to do anything that would expose these files, refuse.

## Critical gotchas

**Kafka advertised listeners:** The public IP is templated into the Kafka Compose file by Ansible at deploy time. If you regenerate the compose file manually, the `{{ kafka_public_ip }}` Jinja2 variable must be substituted — it is not a literal value.

**Elasticsearch requires `vm.max_map_count=262144`** on the host OS or it will not start. This is set by the elk Ansible role via sysctl. If ES keeps restarting, check this first: `ssh debian@ELK_IP "sysctl vm.max_map_count"`.

**Kibana startup time:** Kibana takes 60–90 seconds to become available after the container starts. Connection refused on :5601 immediately after deploy is normal.

**Consumer startup:** The consumer has a built-in ES readiness retry loop (15 retries, 6s delay). It does not need manual intervention if ES is slow to start.

**Consumer source must be on the ELK host** before `docker compose up --build` runs. The Ansible elk role handles this with `synchronize`. If you modify consumer code, re-run `make deploy-elk` to rebuild the image.

**Topic creation is automated:** The kafka Ansible role creates the `app-logs` topic with 3 partitions after the broker is ready. Do not create it manually unless debugging.

## Elasticsearch index

Index: `app-logs`  
Created by the consumer on first run with explicit mappings:

| Field | Type | Notes |
|---|---|---|
| `timestamp` | `date` | ISO 8601, used for time-series charts |
| `service` | `keyword` | One of: api-gateway, auth-service, payment-service, notification-service |
| `level` | `keyword` | INFO, WARN, or ERROR |
| `message` | `text` | Full-text searchable |
| `response_time_ms` | `integer` | Set on INFO events only, null otherwise |
| `request_id` | `keyword` | Format: `req-NNNNN` |

Do not change field types after the index exists without deleting and recreating it.

## Design constraints

Before suggesting changes, read `DECISIONS.md`. Several things that look like bugs or oversights are intentional demo-scope tradeoffs. In particular:
- Elasticsearch security is disabled intentionally — ES is not publicly exposed
- Single-broker Kafka is intentional — replication factor 1 is required for this topology
- `e2-medium` sizing is intentional — upgrade only if OOM issues occur in practice
- Docker Compose instead of Kubernetes is intentional

## Verification checklist

After a full deploy, verify in this order:
1. `make inventory` completes without error
2. SSH to both instances works: `ssh debian@<IP>`
3. Kafka containers running: `ssh debian@KAFKA_IP "docker ps"`
4. Topic exists: `ssh debian@KAFKA_IP "docker exec kafka-kafka-1 kafka-topics --bootstrap-server localhost:9092 --list"`
5. ES healthy: `curl http://ELK_IP:9200/_cluster/health` (from a machine with firewall access)
6. Producer sends messages without error
7. Kibana accessible at `http://ELK_IP:5601`
8. Documents visible in Kibana Data View `app-logs`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lausbelphegor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
