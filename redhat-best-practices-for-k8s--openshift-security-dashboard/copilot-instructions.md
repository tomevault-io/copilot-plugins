## openshift-security-dashboard

> This file provides guidance to AI coding agents when working with this repository.

# AGENTS.md

This file provides guidance to AI coding agents when working with this repository.

## Overview

OpenShift Security Dashboard is a Next.js web application for exploring and managing the security posture of an OpenShift or Kubernetes cluster. Users upload a kubeconfig file to visually analyze RBAC, Security Context Constraints (SCCs), Network Policies, Pod Security Admission, Workloads, Service Accounts, Secrets, OAuth/Identity, and TLS certificates -- all from the browser with built-in tooltips and an optional local AI assistant.

## Technology Stack

- **Next.js 16** (App Router, Turbopack) -- Fullstack React framework
- **React 19** -- UI library
- **TypeScript 5** -- Type-safe JavaScript
- **Tailwind CSS 4** -- Utility-first CSS framework
- **shadcn/ui** + **Radix UI** -- Accessible, composable UI components
- **@kubernetes/client-node** -- Official Kubernetes JavaScript client (server-side only)
- **@xyflow/react** (React Flow) -- Interactive node-based graph visualization
- **Zustand** -- Lightweight client-side state management
- **Zod** -- Schema validation
- **Lucide React** -- Icon library
- **Ollama** -- Local LLM inference for the AI assistant (optional)

## Prerequisites

- **Node.js**: 18+ LTS required
- **npm**: 9+ (ships with Node.js)
- **Ollama**: Optional, only needed for the AI assistant feature (`ollama pull qwen3:32b`)

## Commands

| Command | Description |
|---|---|
| `npm install` | Install all dependencies |
| `npm run dev` | Start development server with Turbopack (http://localhost:3000) |
| `npm run build` | Create a production build |
| `npm start` | Serve the production build |
| `npm run lint` | Run ESLint across the codebase |

## Code Structure

```
src/
  app/                        Next.js App Router pages and API routes
    api/                      Server-side API route handlers
      ai/                    Ollama proxy for AI assistant
      changes/               Apply staged changes to cluster
      cluster/               Cluster info endpoint
      identity/              OAuth / Identity (OpenShift)
      kubeconfig/            Upload, parse, switch kubeconfig
      labels/                Label queries
      network-policies/      Network policy data
      pod-security/          Pod Security Admission labels
      rbac/                  Roles, ClusterRoles, Bindings
      scc/                   Security Context Constraints
      scc-associations/      Unified SCC-to-SA resolution
      secrets/               Secret metadata (values never exposed)
      service-accounts/      Service account enrichment
      tls-scan/              TLS scanning (node-level port + cipher enumeration)
      workloads/             Deployments, StatefulSets, Pods, etc.
    analysis/                 Analysis page (PSA compliance, cross-namespace)
    identity/                 OAuth & Identity page
    network/                  Network Policies page
    pod-security/             Pod Security Admission page
    rbac/                     RBAC Explorer page
    scc/                      SCC Manager page
    secrets/                  Secrets page
    service-accounts/         Service Accounts page
    tls/                      TLS Compliance page
    workloads/                Workloads page
    layout.tsx                Root layout
    page.tsx                  Landing / Dashboard page
    globals.css               Global styles

  components/
    ai/                       Chat panel, AI suggestion renderer
    analysis/                 PSA compliance, cross-namespace matrix, top risks
    dashboard/                Security score, domain cards
    layout/                   Sidebar, header, resource navigator
    network/                  Network topology, policy detail, label picker
    rbac/                     RBAC graph, matrix, who-can, role/binding detail
    scc/                      SCC comparison, detail, relationships
    secrets/                  Secret detail
    service-accounts/         SA detail
    shared/                   FieldLabel, LearnTooltip, ResourceLink, ResourceDetailSheet
    tls/                      TLS scan upload, results table, port detail, summary
    ui/                       shadcn/ui primitives (button, card, dialog, etc.)
    wizards/                  Guided workflow wizards
    workloads/                Workload list, pod detail, security context editor

  lib/
    ai/                       AI assistant utilities
    k8s/                      Kubernetes API client, cache, data fetchers
      client.ts              K8s client initialization from uploaded kubeconfig
      cache.ts               Server-side response caching
      rbac.ts                RBAC data fetcher (roles, bindings, subjects)
      scc.ts                 SCC data fetcher
      scc-rbac.ts            SCC-to-RBAC resolver (direct, group, RBAC-based)
      network.ts             Network policy data fetcher
      pod-security.ts        Pod Security Admission fetcher
      workloads.ts           Workload fetcher (all workload types)
      service-accounts.ts    Service account fetcher with SCC enrichment
      secrets.ts             Secret metadata fetcher
      identity.ts            OAuth/Identity fetcher (OpenShift)
      tls-scanner.ts         TLS scanner (node-level via oc debug)
      node-scan-script.ts    Bash scan script for node-level TLS probing
      ports.ts               Port analysis utilities
    risk/                     Risk scoring engine
      rules.ts               Risk rule definitions
      scoring.ts             Risk score calculator
      psa-compliance.ts      PSA compliance checker
      cross-namespace.ts     Cross-namespace access analysis
    glossary.ts               60+ Kubernetes/OpenShift field-level term definitions
    utils.ts                  General utility functions

  store/
    cluster-store.ts          Cluster connection state (context, namespace, kubeconfig)
    changes-store.ts          Staged changes state (pending modifications)

  types/
    index.ts                  TypeScript type definitions for all domain objects
```

## Features

- **Dashboard** -- Security posture score, top risk findings, domain quick-stats
- **RBAC Explorer** -- Interactive graph of Subject-Binding-Role-Resource chains, permission matrix, "Who Can?" queries
- **SCC Manager** -- Side-by-side comparison of Security Context Constraints with traffic-light risk indicators; full editing support
- **Network Policies** -- Topology visualization, namespace-level segmentation, policy creation by drawing links
- **Pod Security Admission** -- Namespace grid showing enforce/audit/warn levels with comparison and editing
- **Workloads** -- All workload types with per-container security context editing, port analysis, applied/available SCCs
- **Service Accounts** -- Permission rollup, cluster-admin detection, linked secrets/roles, unified SCC resolution
- **Secrets** -- Metadata-only view categorized by type with age/rotation warnings (values never displayed)
- **OAuth & Identity** -- Identity providers, OAuth clients, users, and groups browser
- **TLS Compliance** -- Node-level TLS scanning producing openshift/tls-scanner compatible output
- **Analysis** -- PSA compliance checker, cross-namespace access matrix, risk scoring
- **AI Assistant** -- Ollama-powered contextual assistant on every resource detail page (optional)
- **Change Staging** -- All modifications are previewed before applying; staged changes can be reviewed, edited, or discarded

## Architecture Notes

- The kubeconfig is processed entirely on the local machine and is only sent to the local Next.js server process (localhost)
- `@kubernetes/client-node` is server-side only (configured via `serverExternalPackages` in `next.config.ts`)
- The AI assistant runs locally via Ollama -- no data is sent to cloud APIs
- Secret values are never displayed in the browser -- only metadata (names, types, keys, age)
- All cluster modifications go through the change staging system for review before applying

## Key Dependencies

| Package | Purpose |
|---|---|
| `@kubernetes/client-node` | Official Kubernetes JavaScript client for cluster API access |
| `@xyflow/react` | Interactive node-based graph visualization (RBAC graph, network topology) |
| `zustand` | Lightweight client-side state management |
| `zod` | Runtime schema validation |
| `radix-ui` / `shadcn` | Accessible, composable UI component primitives |
| `cmdk` | Command palette (Cmd+K) interface |
| `next-themes` | Dark/light theme support |
| `yaml` | YAML parsing for kubeconfig files |
| `sonner` | Toast notification system |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/redhat-best-practices-for-k8s)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/redhat-best-practices-for-k8s)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
