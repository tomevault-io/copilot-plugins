## dl-team-proposal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Deep Learning Team Proposal** repository. The project objective is to **train a language model using nanochat, then create multiple model variants to produce a learning curve comparison** studying which techniques (e.g., SFT, RLHF, depth/hyperparameter configurations) contribute most to model improvement. Background research also covers **nanoGPT** and **llm.c** (all by Andrej Karpathy) for comparative context.

## Repository Structure

- `docs/James_notes/` — James's research notes (numbered 01-07, with 00 as table of contents)
- `README.md` — Project overview
- `LICENSE` — MIT License

## Current State

This is a **research-only repository** — no code implementation. Work products are documentation and comparative analysis of the three frameworks being evaluated:

| Framework | Language | Scope | Status |
|-----------|----------|-------|--------|
| nanoGPT | Python/PyTorch | Pretraining only | Deprecated |
| llm.c | C/CUDA | Pretraining only | Active |
| nanochat | Python/PyTorch | Full pipeline (pretrain + SFT + RLHF + chat UI) | Active, successor to nanoGPT |

## Key Context

- nanochat is the official successor to nanoGPT (released Oct 2025)
- The team has an existing comparison table covering language, dependencies, performance, parameters, training stages, cost, and ease of use across all three frameworks
- llm.c offers ~7% faster performance than PyTorch with minimal dependencies (pure C/CUDA)
- nanochat introduces a "Complexity Dial" (`--depth` parameter) that auto-configures all hyperparameters

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jamesEmerson112) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
