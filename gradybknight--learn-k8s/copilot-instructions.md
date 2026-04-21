## learn-k8s

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Kubernetes learning repository focused on setting up and administering a 3-node Kubernetes cluster using physical hardware. The project follows a structured learning path from Linux installation to full GitOps deployment.

## Hardware Architecture

- **Control Plane Node**: Lenovo ThinkCentre M720Q (Intel i5-8500T, 16GB RAM, 256GB SSD)
- **Worker Nodes**: 2x Beelink Mini S12 Pro (Intel N100, 16GB DDR4, 500GB M.2 SSD)
- **Network**: All nodes connected via switch → router on Google Fiber

## Learning Progression

The repository follows a structured syllabus:

1. **Linux Server Setup** - OS selection, ISO creation, individual machine configuration, network setup
2. **Kubernetes Installation** - K8s software installation with Lenovo as primary control plane
3. **Backend Application** - Node/TypeScript app with `/health` and `/hello` endpoints, database integration
4. **Frontend Application** - React/TypeScript SPA consuming backend APIs
5. **External Access** - Exposing cluster services without static IP constraints
6. **GitOps Implementation** - CI/CD automation, ArgoCD evaluation

## Development Structure

This is a learning-focused repository designed for hands-on Kubernetes cluster administration. All documentation should be structured as step-by-step markdown instructions that can be followed sequentially.

## Key Considerations

- Focus on practical, hands-on learning rather than production-ready configurations
- Document each step thoroughly for educational purposes
- Consider home network limitations (no static IP guarantee)
- Plan for physical hardware constraints and networking setup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gradybknight) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
