## runtimes-benchmarks

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Kubernetes-based benchmarking framework for running Platformatic Watt performance tests on Amazon EKS (Elastic Kubernetes Service). The benchmark compares application performance when running with Node.js (single-thread), Bun (single-thread), and Platformatic Watt (multi-thread) using Next.js.

## Architecture

The repository follows this structure:

1. **Common Functions Library** (`lib/common.sh`) - Shared bash functions for logging, tool validation, HTTP health checks, and cleanup
2. **Next.js Application** (`next/`) - The demo application used for benchmarking
3. **EKS Orchestration** (`benchmark.sh`) - Main benchmarking script that creates AWS infrastructure, deploys to EKS, runs load tests, and cleans up

### Key Design Decisions

- **Pure AWS CLI**: Creates all infrastructure using AWS CLI (no eksctl, CloudFormation, or Terraform)
- **Local Docker Build + ECR**: Images are built locally and pushed to an ephemeral ECR repository (no external CI/CD dependency)
- **AWS NLB LoadBalancer Services**: Services use LoadBalancer type with AWS Network Load Balancer (internal scheme)
  - Services annotated with `benchmark.platformatic.dev/expose: "true"` are discovered and benchmarked
  - LoadBalancer hostnames are passed to load test instance via URL_NODE, URL_BUN, URL_WATT environment variables
- **Separate Load Testing Instance**: k6 runs on a dedicated EC2 instance (not in the cluster) to simulate realistic network conditions and avoid resource contention
- **Automatic Cleanup**: All resources (cluster, node group, VPC, IAM roles, EC2 instance, ECR repository, Load Balancers) are cleaned up via trap handlers on exit/failure

### Infrastructure Flow

1. Creates ECR repository and builds/pushes Docker image locally
2. Creates VPC with subnets, internet gateway, and route tables
3. Creates IAM roles for cluster and nodes
4. Creates EKS cluster and managed node group
5. Configures kubectl context (cluster name)
6. Deploys Next.js application from `kube.yaml` (templated with ECR image URI)
7. Waits for pods to be ready using kubectl
8. Discovers annotated LoadBalancer services
9. Waits for AWS NLB hostnames to be assigned
10. Launches EC2 instance running k6 load tests with LoadBalancer URLs
11. Monitors console output and displays results
12. Cleans up all resources (including ECR repository and Load Balancers)

## Running Benchmarks

### Prerequisites

Before running benchmarks, ensure:
- Docker is installed and running
- AWS CLI v2 is installed and configured with a default region
- kubectl is installed
- jq is installed
- AWS profile has required permissions (see `lib/minimum-policy.json`)

Apply required AWS IAM permissions:
```sh
AWS_PROFILE=<profile-name> ./setup-policy.sh
```

### Execute Benchmark

Run the main benchmark script:
```sh
AWS_PROFILE=<profile-name> ./benchmark.sh
```

The script will:
- Build Docker image locally and push to ECR
- Create an EKS cluster (takes 15-20 minutes)
- Deploy the Next.js app with three variants (Node, Bun, Watt)
- Launch EC2 instance running k6 load tests
- Display performance results
- Clean up all resources automatically (including ECR repository)

### Environment Variables

Required:
- `AWS_PROFILE` - AWS CLI profile to use

Optional (with defaults):
- `CLUSTER_NAME` - EKS cluster name (default: `watt-benchmark-<timestamp>`)
- `NODE_TYPE` - EC2 instance type for EKS nodes (default: `m5.2xlarge`)
- `NODE_COUNT` - Number of worker nodes (default: `3`)
- `AMI_ID` - Amazon Linux 2023 AMI for load testing EC2 (default: `ami-07b2b18045edffe90`)
- `LOADTESTING_INSTANCE_TYPE` - EC2 instance type for k6 (default: `c7gn.large`)
- `ECR_REPO_NAME` - ECR repository name (default: `watt-benchmark`)
- `IMAGE_TAG` - Docker image tag (default: `latest`)

## Next.js Application Structure

The `next/` directory contains:

### Files
- `Dockerfile` - Docker image build configuration (includes both Node.js and Bun runtimes)
- `entrypoint.sh` - Docker entrypoint that runs the appropriate runtime based on SCRIPT_NAME
- `kube.yaml` - Kubernetes manifests for three deployment variants
- `loadtest.sh` - k6 load testing script
- `watt.json` - Platformatic Watt configuration
- `package.json` - NPM scripts and dependencies

### Package Scripts
- `start:node` - Run with standalone Node.js server (single-thread)
- `start:bun` - Run with Bun runtime (single-thread)
- `start:watt` - Run with Platformatic Watt (2 workers by default)

### Kubernetes Deployment (`kube.yaml`)
- Three Deployments: `next` (Node), `next-bun`, `next-watt`
- Three Services with LoadBalancer type (AWS NLB) and benchmark annotation
- Environment variables control which script runs (`SCRIPT_NAME`, `WORKERS`)
- Image reference uses `IMAGE_PLACEHOLDER`, templated at deploy time with ECR URI
- topologySpreadConstraints for even pod distribution across nodes

### Docker Image (`Dockerfile`)
- Based on Node 24 Alpine with Bun installed
- Builds app during image build
- `entrypoint.sh` detects SCRIPT_NAME and runs appropriate runtime

### Local Development

```sh
cd next
npm install
npm run dev      # Development server with hot reload
npm run build    # Build Next.js app
```

### Load Testing

The `loadtest.sh` script runs k6 load tests sequentially against all three services:
- 1000 requests/second for 120 seconds per service
- 480 second cooldown between tests
- Tests run on separate EC2 instance within same VPC
- Uses URL_NODE, URL_BUN, URL_WATT environment variables for LoadBalancer endpoints

## Common Functions (lib/common.sh)

Shared utilities used by benchmark script:

- **Logging**: `log()`, `error()`, `success()`, `warning()` - Colored output functions
- **Validation**: `check_tool()`, `validate_aws_tools()`, `validate_common_tools()`, `validate_required_vars()`
- **Health Checks**: `wait_for_http()` - Polls HTTP endpoint until ready
- **Cleanup**: `generic_cleanup()` - Calls provider-specific `cleanup_instances()`

## Code Style

- **Shell Scripts**: Bash with `set -e`, use common.sh logging functions
- **AWS Operations**: Use `--profile "$AWS_PROFILE"` for all AWS CLI commands
- **Error Handling**: Trap handlers for cleanup, `|| true` for best-effort cleanup operations
- **Quiet Mode**: Redirect verbose output to `/dev/null` or filter with grep

---
> Source: [platformatic/runtimes-benchmarks](https://github.com/platformatic/runtimes-benchmarks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
