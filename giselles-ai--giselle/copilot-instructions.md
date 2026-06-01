## vercel-cli

> Setup Vercel CLI


# Vercel CLI Setup Guide

## Installation

To install Vercel CLI globally using pnpm (recommended for this project):

```bash
pnpm i -g vercel
```

## Authentication


Log in to your Vercel account:

```bash
vercel login
```

## Project Setup

### Link to Existing Project

```bash
vercel link
```

## Development Workflow

### Test Locally

Run a local development server that replicates the Vercel environment:

```bash
vercel dev
```

## Environment Variables

### Pull Environment Variables

Download environment variables to a local `.env` file:

```bash
vercel env pull
```

## Common Commands

### Check CLI Version

```bash
vercel --version
```

### Update CLI

```bash
pnpm i -g vercel@latest
```

### View Deployments

```bash
vercel list
```

### View Logs

```bash
vercel logs [deployment-url]
```

## Project Configuration

Vercel projects can be configured using a `vercel.json` file in your project root. This allows you to define build settings, routes, environment variables, and more.

Refer to the [Vercel documentation](mdc:https:/vercel.com/docs/cli) for more detailed information on available commands and configuration options.

---
> Source: [giselles-ai/giselle](https://github.com/giselles-ai/giselle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
