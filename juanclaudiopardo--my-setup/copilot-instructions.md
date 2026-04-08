## my-setup

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

Personal documentation repo for Claude Code configuration. Contains installation references for MCP servers and skills — no application code, build system, or tests.

Language: Spanish (all documentation is written in Spanish).

## Structure

- `mcp/README.md` — MCP server installation commands (Playwright, shadcn/ui, Figma, Next.js DevTools, TanStack, Nx)
- `skills/README.md` — Skills CLI usage and installed skills organized by category (React/Next.js, AI SDK, Design/UI, Browser Automation, Deployment, Commerce, Workflow, JSON Render, Testing, SEO, Nx), plus AGENTS.md guidance

## Key References

- Skills CLI: `npx skills@latest` (add, list, find, check, update, remove, init)
- AGENTS.md generation for Next.js projects: `npx @next/codemod@canary agents-md`
- Nx AI agents setup (skills + MCP + CLAUDE.md): `npx nx configure-ai-agents`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/juanclaudiopardo)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/juanclaudiopardo)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
